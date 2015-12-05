title: 使用 Tesseract 识别字符
date: 2015-11-16 15:53:47
tags: OCR
categories: 开发笔记
description: 调教调教，调教出属于你的文字识别系统。
---

最近在做一个微信公众账号，很多地方需要用到字符识别([OCR](https://en.wikipedia.org/wiki/Optical_character_recognition))。补了补一些基础知识之后，决定基于 [Tesseract](https://github.com/tesseract-ocr/tesseract/) 进行开发。

## [Tesseract](https://github.com/tesseract-ocr/tesseract)

### Install

训练过程是在 Mac 上完成，然后把训练结果扔到 Docker 上。所以需要安装 Mac 和 Linux 两个环境下的 Tesseract 。

首先是 Mac 下，[homebrew](https://github.com/Homebrew/homebrew/blob/master/Library/Formula/tesseract.rb) 以前需要手动编译，最近终于加上了 `--with-training-tools` 的选项，各种训练工具都已经打包搞好，只需要：

```
brew install tessreact --with-training-tools
```

然后 Linux 下，通过 Docker 配置了 Ubuntu@14.04 ，安装也很方便：

```
sudo apt-get install tesseract-ocr
```

### Test

安装完成之后，在没有训练之前先先试一下效果如何。比如下面这张：

![](http://ww4.sinaimg.cn/large/61d238c7jw1ey2y6v8zfzj209q02qglo.jpg)

通过 Tesseract 识别：

```
tesseract input.jpg result
```

识别结果是：

```
08F1916711‘
```
嗯大体上数字识别还是准确的，但是对于一些相近的字符比较容易出现识别错误，例如图中的 G 被识别成了 0。原因是默认会使用英语进行识别，而英语字体集中并没有我们想要识别的字体。

这时候我们需要训练自己的字体集，在后面的例子中，我们以 `Mourney` 字体为例。（并没有这个字体，这只是我们项目的行动代号）

### Training

准备好用来训练的图片文件和用来测试效果的图片文件：

```
├── mon.mourney.exp0.tif
└── test.jpg
```

首先，用 `tessreact` 命令识别原图片，制作 box 文件：

```
tesseract mon.mourney.exp0.tif mon.mourney.exp0 batch.nochop makebox
```

会发现目录中多了 `mon.mourney.exp0.box` 文件：

```
├── mon.mourney.exp0.box
├── mon.mourney.exp0.tif
└── input.jpg
```

box 文件的内容很简单，就是告诉 Tesseract 这张图里有哪些字符，以及它们的位置，相当于标记自定义字体的包围盒。文件内容大概是这样的：

```
G 32 1384 49 1411 0
8 56 1384 74 1412 0
F 83 1382 103 1413 0
```

文中数字分别对应图片中文字『左下右上』的坐标。

然后下载 [jTessBoxEditor](http://vietocr.sourceforge.net/training.html) ，这是一个第三方的开源编辑器，可以直接编辑 box 文件。点击 Box Editor ，然后打开原图，就可以在可视化界面里编辑调整 box 的位置。

编辑完成后，执行以下命令（具体每条命令的含义请参见[官方文档](https://github.com/tesseract-ocr/tesseract/wiki/TrainingTesseract)）：

```
echo "font 0 0 0 0 0" > font_properties
tesseract mon.mourney.exp0.tif mon.mourney.exp0 nobatch box.train
unicharset_extractor mon.mourney.exp0.box
mftraining -F font_properties -U unicharset -O mon.unicharset mon.mourney.exp0.tr
cntraining mon.mourney.exp0.tr

mv normproto mon.normproto
mv inttemp mon.inttemp  
mv pffmtable mon.pffmtable  
mv shapetable mon.shapetable  

combine_tessdata mon.
```

执行后注意一下最后输出的 1、3、4、5、13 行，需要确定 Offset 值不是 -1 。训练完成。

此时的文件目录：

```
├── mon.inttemp
├── mon.mourney.exp0.box
├── mon.mourney.exp0.tif
├── mon.mourney.exp0.tr
├── mon.mourney.exp0.txt
├── mon.normproto
├── mon.pffmtable
├── mon.shapetable
├── mon.traineddata
├── mon.unicharset
├── font_properties
├── test.jpg
└── unicharset
```

生成的 `mon.traineddata` 就是我们需要的训练结果，把它移到 Tesseract 的 `tessdata` 目录下，然后再次用 `tesseract` 命令识别，不过这次指定语言为我们定义的 `mon` ：

```
tesseract -l mon test.jpg result
```

这时会发现，已经能够准确的识别出结果了：

```
> Test  cat result.txt
G8F19167111
```

好吧并没有，多识别了一个 1 。原始图片还是需要做一些简单的处理，比如做一下腐蚀、膨胀、二值化等等。

## NodeJS

接下来就是写个后端接口来调用这个 `tesseract` 方法。有了 [multer](https://github.com/expressjs/multer) 这个库来处理文件上传，以及 [node-tesseract](https://github.com/desmondmorris/node-tesseract) 这个库来封装 `tesseract` 命令，代码十分简单：

```js
var fs = require('fs');
var multer = require('multer')
var upload = multer({
  dest: 'uploads/'
})
var express = require('express');
var app = express();
var tesseract = require('node-tesseract');

app.post('/mourney', upload.single('image'), function(req, res, next) {
  var image = req.file;
  var options = {
    l: 'mon',
    psm: 7
  };
  tesseract.process(image.path, options, function(err, text) {
    console.log(text);
    if (err) {
      console.error(err);
    } else {
      res.send(text);
    }
  });
});

var server = app.listen(3000, function() {
  var host = server.address().address;
  var port = server.address().port;

  console.log('listening at http://%s:%s', host, port);
});
```

部署到服务器上，用 `Paw` 测试一下：

![](http://ww4.sinaimg.cn/large/61d238c7gw1ey5c36wusej21iw0e0di5.jpg)

成功，后端初步调试完成。


***

参考文献：

- [Training Tesseract](https://github.com/tesseract-ocr/tesseract/wiki/TrainingTesseract)
