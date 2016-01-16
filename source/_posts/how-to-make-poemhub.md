title: PoemHub 小记：如何随机生成唐诗宋词
date: 2016-01-12 10:21:08
tags: JS
categories: 开发笔记
description: 最近一周，做的最多的事情就是，鞭尸，啊不，编诗
---

最近和学生团队一起完成了一款玩票产品：[《生日诗词》](http://lemur-wechat-fe.leanapp.cn/poetry/index.html?openid=oFzyEv0NTD9kuizJrYNoIUCAVhyE)。功能很简单：根据你的生日生成一首诗词。

本以为扔一堆高频词组然后随机组装就行了，然而实际做起来却发现其乐无穷。

## 需求分析

既然是唐诗宋词，那肯定有一定的格式。五言七言、绝句律诗，后面说不定还要支持宋词元曲，如何才能灵活的生成想要的格式呢？

先来看个五言绝句：

```
平平仄仄平（韵）
仄仄仄平平（韵）
仄仄平平仄
平平仄仄平（韵）
```

可以看到，主要需要关注的是平仄和韵脚。如果能自己定义模板，然后再攒一大堆唐诗宋词通用的词组，最后将数据填充到模板里就好了。

## 前期准备

### 自定模板

模板的话大致需要一下几个信息：

- 字数：希望用多少字的词组来填充
- 平仄：朗朗上口的话平仄是必须的
- 押韵：合适的韵脚可以让诗更像诗
- 标记：通用的标记可以更好地渲染

这里补充一下平仄的基础知识：普通话中，第一声、第二声称为阴平、阳平，是由古汉语的平声演变而来。第三声上声、第四声去声，则为仄声。

于是结果几番折腾，确定了一下模板：

- 类型：A/B 区分，A 为不押韵，B 为押韵
- 字数：1 是单字，2 是词组
- 平仄：- 是平，对应阴平、阳平。+ 是仄，对应上声、去声。
- 标记：用 `\` 将各个单位之间隔开，方便模板引擎解析。

比如前面的那个五言绝句，对应的模板就是：

```
A2-/A2+/B1-/，
A2+/A2-/B1-/。
A2+/A2-/A1+/，
A2-/A2+/B1-/。
```

比如卜算子的模板：

```
'A2+/A2-/A1-/，', 'A2+/A2-/B1+/。', 'A2+/A2-/A2+/A1-/，', 'A2+/A2-/B1-/。',
'A2+/A2-/A1-/，', 'A2+/A2-/B1+/。', 'A2+/A2-/A2+/A1-/，', 'A2+/A2-/B1-/。',
```

我们把所有的模板都放在了 [data/templates.js](https://github.com/Lemur/poemhub/blob/master/lib/data/templates.js) 里。

### 收集词组

数据主要是一些唐诗宋词中的高频词组，比如什么『明月』『悲秋』这种。初步计划是手动整理，后面想做成导入海量诗词歌赋，然后自动解析，按照词性分类筛选，扩充词库。

为了和模板对应，数据也需要根据平仄、韵脚、字数进行分类。一开始我是用 `-/v\` 来区分汉语拼音中的一声二声三声四声的，后来发现没必要区分这么细，只要按照平仄来分组就行。

所以很快就可以整理出一个 A 组：

```js
// A 组
'A': {
  // 一个字
  '1': {
    // 平声
    '-': [
      '东', '屋', '冬', '江', '支', '微', '佳', '灰', '蒸', '庚',
    ],
    // 仄声
    '+': [
      '董', '讲', '纸', '尾', '语', '巧', '雪', '感', '送', '宋',
    ],
...
```

在整理 B 组的时候，因为 B 组是需要押韵的，而同一个韵的词应该放在一个组里，然后在随机生成的时候，随机选择一个韵脚。

所以 B 的结构和 A 稍有些变化，在外面套了个数组，同一个数组里的都是押韵的词组：

```js
// B 组
'B': [
  // dong
  {
    // 一个字
    '1': {
      // 平声
      '-': [
        '东', '空', '松', '通', '庸', '宫', '宗', '荣', '同', '浓',
      ],
      // 仄声
      '+': [
        '涌', '耸', '哄', '拢', '总', '统', '送', '诵', '用', '动',
      ],
...
```

我们把所有的词组都放在了 [data/words.js](https://github.com/Lemur/poemhub/blob/master/lib/data/words.js) 里。

### 可选配置

在进行渲染之前，我们还需要考虑一下可供选择的生成配置。

生成诗歌主要有一下几种模式：

- 随机生成诗歌
- 根据模板生成诗歌
- 根据生日生成诗歌

在实际的代码中，使用 `poem.js` 作为统一的入口， `options` 作为可选配置，然后根据不用的选项，选择不同的 `builder` 进行生成：

```js
function poem(options) {
  // 有生日，进入生日模式
  if (options && options.birthday) {
    return birthdayBuilder(options.birthday)
  }
  // 有模板，进入模板模式
  if (options && options.template) {
    return randomBuilder(options.template)
  }
  // 没有配置信息，随机生成一个
  return randomBuilder()
}
```

## 模板渲染

有了模板有了数据，接下来就是如何渲染模板了。

### 选取模板

如果是随机生成诗词，模板是随机从模板库中选取。我们可以写一个随机函数用于从数组中随机获取其中一个元素：

```js
function getRandomItem(array) {
  return array[Math.floor(Math.random() * (array.length))]
}
```

如果是生日生成诗词，则套用公式： `templates[(year+month) % templates.length]`。这公式没啥缘由，只是觉得同年同月实属难得，那么共用一个模板也是缘分呐。

### 选取数据

由于各个诗句结构不用，所以不同类型的数据源数目肯定是多余的。如何从大量数据源中选择自己需要的数据供模板渲染呢？

首先，需要知道当前模板需要哪些类型的数据，以及具体的数量。对原始模板做一次 `reduce` 将句子的数组合并成单元的数组，然后再用一次 `reduce` 将单元数组的内容进行统计，生成统计结果：

```js
function countTemplate(template) {
  return template
    .reduce((pv, cv) => {
      const words = cv.split('/')
      words.pop()
      return pv.concat(words)
    }, [])
    .reduce((pv, cv) => {
      const v = pv[cv] || 0
      pv[cv] = v + 1
      return pv
    }, {})
}
```

统计计数的结果是这样的：

```js
// 6个仄声不押韵词组，6个平声不押韵词组，3个平声押韵单字，1个仄声押韵单字
{ 'A2+': 6, 'A2-': 6, 'B1-': 3, 'A1+': 1 }
```

统计完了之后再去选择数据就会比较简单了，任务就变成了『从100个仄声不押韵词组中选择6个』这样的。

然而在随机选取的时候还是会有问题，比如『西风』和『西湖』这种，一般来说出现两个相同的字感觉会比较不顺，所以还需要做一次去重。

我们通过 `buildTemplateArray` 这个函数进行去重并生成数据，第一个参数是原始数据，比如所有的 A2+ 词组；第二个参数是需要的数目；第三个参数是当前的统计结果，用于去重。

函数代码如下：

```js
function buildTemplateArray(array, count, cTempData) {
  const resultArray = []
  while (resultArray.length !== count) {
    // 随机生成汉字词组
    var r = getRandomItem(array)
    // 随机选择的词组和现有的词组中有多少字是相同的
    var sameWords = Object.keys(cTempData)
      .reduce((pv, cv) => {
        return pv.concat(cTempData[cv].join('').split(''))
      }, [])
      .concat(resultArray.join('').split(''))
      .filter((cWord) => {
        var same = false
        r.split('').forEach((w) => {
          if (w == cWord) {
            same = true
          }
        })
        return same
      })
    //  如果随机结果和已有数据重复，则弃用本次生成结果
    if (sameWords.length === 0) {
      resultArray.push(r)
    }
  }
  return resultArray
}
```

选取完成的数据是这样的：

```
{ 'A2+': [ '憔悴', '天气', '何事', '桃李', '芳草', '风雨' ],
  'A2-': [ '西风', '相思', '谁知', '长安', '东君', '而今' ],
  'A1+': [ '树', '路' ],
  'B1-': [ '同', '浓' ] }
```

### 填充数据

模板也选好了，数据也选好了，接下来就是把数据填充到模板里。填充的过程通过 `render` 函数完成：

```js
function render(template, templateData) {
  const result = template
    .reduce((pv, cv) => {
      // 通过 / 将句子分割成基础单元
      const words = cv.split('/')
      // 取出句末的标点符号
      const punctuation = words.pop()
      // 对剩下来的诗句部分进行替换，从所属类型的数据源中取出第一个元素
      const r = words.map((placeholder) => {
        const w = templateData[placeholder].pop()
        return w
      })
      // 拼接并返回结果
      return pv + r.join('') + punctuation + '\n'
    }, '')
  return result
}
```

随便测试一下生成的结果：

```
西湖风月斜阳刚，
富贵平生憔悴亡。
桃李而今千里御，
江南心事无人扬。

今夜艳思乡，
多少漫疆。
当时惟有斜阳杭。
何事风流芳草让。
不是轻扬。
```

虽然平仄、字数都是对的，但是，每句话都是生拼硬凑的，没有什么真实的含义。

## 进阶

再接下来就是如何让诗句富有含义，因为目前词库还是手动整理的，并不是很丰富，如果后面有时间可以做一个自动导入工具，用爬虫爬取大量唐诗宋词，然后用词法分析工具将内容进行分析归类，然后后面再拼接诗句的时候用一些常见的词法逻辑进行拼接，比如『名词名词形容词』这种。

刚好在微博上看到清华自然语言处理实验室开源的一个中文词法分析工具包：[THULAC](http://thulac.thunlp.org/)

## 小结

毕竟是一款玩票产品，不过没想到还是花了很多精力。主要是通过玩票试一试以前接触的一些知识，比如用 [tape](https://github.com/substack/tape) 写测试，比如用 [babel](https://github.com/babel/babel) 将 ES6 的语法转换成 ES5 的语法。

后面如果有时间再试试词法分析工具。

感觉还是很有意思的～

完整项目可以点击这里：[poemhub](https://github.com/Lemur/poemhub)

***
参考资料：

- [维基百科：平仄](https://zh.wikipedia.org/wiki/%E5%B9%B3%E4%BB%84)
