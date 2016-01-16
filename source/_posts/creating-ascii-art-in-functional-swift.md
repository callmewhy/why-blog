title: 用函数式的 Swift 实现图片转字符画的功能
date: 2015-06-07 16:54:43
tags: Swift
categories: 开发笔记
description: 如何更 Functional 地写 Swift 代码，希望此文可以带来启发。
---

今天整理 Pocket 中待看的文章，看到这篇《[Creating ASCII art in functional Swift](http://ijoshsmith.com/2015/04/29/creating-ascii-art-in-functional-swift/)》，讲解如何用 Swift 将图片转成 ASCII 字符。具体原理文中讲解的很详细，不再赘述，但是标题中的 **in functional Swift** 让我很感兴趣，想知道 `functional` 到底体现在哪里，于是下载 [swift-ascii-art](https://github.com/ijoshsmith/swift-ascii-art) 源码一探究竟。


## Pixel

图片是由各个像素点组成的，在代码中像素通过 `Pixel` 这个 `struct` 实现。每个像素分配了4个字节，这4个字节 (2^8 = 256) 分别用来存储 RBGA 的值。

### createPixelMatrix

可以通过 `createPixelMatrix` 这个静态方法创建一个 `width` * `height` 像素矩阵：

```swift
    static func createPixelMatrix(width: Int, _ height: Int) -> [[Pixel]] {
        return map(0..<height) { row in
            map(0..<width) { col in
                let offset = (width * row + col) * Pixel.bytesPerPixel
                return Pixel(offset)
            }
        }
    }
```

和传统方法中使用 `for` 循环来创建多维数组有所不同的是，这里是通过 `map` 函数实现的。在 Swift 2.0 中， `map` 函数已经被干掉了，只能作为方法调用。


### intensityFromPixelPointer

```swift
`intensityFromPixelPointer` 方法计算并返回像素点的亮度值，代码如下：

    func intensityFromPixelPointer(pointer: PixelPointer) -> Double {
        let
        red   = pointer[offset + 0],
        green = pointer[offset + 1],
        blue  = pointer[offset + 2]
        return Pixel.calculateIntensity(red, green, blue)
    }

    private static func calculateIntensity(r: UInt8, _ g: UInt8, _ b: UInt8) -> Double {
        let
        redWeight   = 0.229,
        greenWeight = 0.587,
        blueWeight  = 0.114,
        weightedMax = 255.0 * redWeight   +
                      255.0 * greenWeight +
                      255.0 * blueWeight,
        weightedSum = Double(r) * redWeight   +
                      Double(g) * greenWeight +
                      Double(b) * blueWeight
        return weightedSum / weightedMax
    }
```


`calculateIntensity` 方法基于 [Y'UV](https://en.wikipedia.org/wiki/Grayscale#Luma_coding_in_video_systems) 编码获取某个像素的亮度 (intensity) ：

> Y' =  0.299 R' + 0.587 G' + 0.114 B'

YUV 是一种颜色编码方法，Y 表示亮度， UV 用来表示色差， U 和 V 是构成彩色的两个分量。它的优点是可以利用人眼的特性来降低数字彩色图像所需要的存储容量。我们通过这个公式获取到的 Y 就是亮度的值。

### Offset

`Pixel` 中其实只存了一个值： `offset` 。 `Pixel.createPixelMatrix` 创建出来的矩阵是这样的：

```
    [[0, 4, 8, ...], ...]
```

并没有像想象中那样存储了每个像素相关数据，而更像是一个转换工具，计算 `PixelPointer` 的灰度值。

## AsciiArtist

`AsciiArtist` 里封装了一些生成字符画的方法。

### createAsciiArt

`createAsciiArt` 方法就是创建字符画：

```swift
    func createAsciiArt() -> String {
        let
        // 加载图片数据，获取指针对象
        dataProvider = CGImageGetDataProvider(image.CGImage),
        pixelData    = CGDataProviderCopyData(dataProvider),
        pixelPointer = CFDataGetBytePtr(pixelData),
        // 将图片转成亮度值矩阵
        intensities  = intensityMatrixFromPixelPointer(pixelPointer),
        // 将亮度值转成对应字符
        symbolMatrix = symbolMatrixFromIntensityMatrix(intensities)
        return join("\n", symbolMatrix)
    }
```

其中 `CFDataGetBytePtr` 函数返回了图像的字节数组指针，数组里每个元素都是一个字节，即 0~255 的整数。每4个字节组成了一个 `Pixel` ，分别对应着 RGBA 的值。

### intensityMatrixFromPixelPointer

`intensityMatrixFromPixelPointer` 这个方法是通过 `PixelPointer` 生成对应的亮度值矩阵：

```swift
    private func intensityMatrixFromPixelPointer(pointer: PixelPointer) -> [[Double]]
    {
        let
        width  = Int(image.size.width),
        height = Int(image.size.height),
        matrix = Pixel.createPixelMatrix(width, height)
        return matrix.map { pixelRow in
            pixelRow.map { pixel in
                pixel.intensityFromPixelPointer(pointer)
            }
        }
    }
```

首先通过 `Pixel.createPixelMatrix` 方法创建了一个空的二维数组，用来存放数值。然后用两个 `map` 嵌套遍历里面的所有元素，将像素 (`pixel`) 转换成亮度 (`intensity`) 的值。

### symbolMatrixFromIntensityMatrix

`symbolMatrixFromIntensityMatrix` 函数将亮度值数组转换成字符画数组：

```swift
    private func symbolMatrixFromIntensityMatrix(matrix: [[Double]]) -> [String]
    {
        return matrix.map { intensityRow in
            intensityRow.reduce("") {
                $0 + self.symbolFromIntensity($1)
            }
        }
    }
```

`map` + `reduce` 成功实现了字符串的累加，每次 `reduce` 都是通过 `symbolFromIntensity` 方法获取到亮度值对应的字符。 `symbolFromIntensity` 方法如下：

```swift
    private func symbolFromIntensity(intensity: Double) -> String
    {
        assert(0.0 <= intensity && intensity <= 1.0)

        let
        factor = palette.symbols.count - 1,
        value  = round(intensity * Double(factor)),
        index  = Int(value)
        return palette.symbols[index]
    }
```

传入 `intensity` ，在确保了值的范围是 0 ~ 1 之后，通过 `AsciiPalette` 将它转换成对应的字符，输出 `sumbol` 。

## AsciiPalette

`AsciiPalette` 是用来将数值转换成字符的工具，像是一个字符画里的调色板一样，根据不同的颜色生成字符。

### loadSymbols

`loadSymbols` 加载了所有的字符：

```swift
    private func loadSymbols() -> [String]
    {
        return symbolsSortedByIntensityForAsciiCodes(32...126) // from ' ' to '~'
    }
```

可以看到，我们选用的字符范围是 32 ~ 126 的字符，接下来就是通过 `symbolsSortedByIntensityForAsciiCodes` 方法将这些字符按照亮度进行排序。比如 `&` 符号肯定代表着比 `.` 暗的区域，那么它是如何比较的呢？请看排序方法。

### symbolsSortedByIntensityForAsciiCodes

`symbolsSortedByIntensityForAsciiCodes` 方法实现了字符串的生成和排序：

```swift
    private func symbolsSortedByIntensityForAsciiCodes(codes: Range<Int>) -> [String]
    {
        let
        // 通过 Ascii 码生成字符数组备用
        symbols          = codes.map { self.symbolFromAsciiCode($0) },
        // 将字符绘制出来，把字符数组转换成图片数组，用于比较亮度
        symbolImages     = symbols.map { UIImage.imageOfSymbol($0, self.font) },
        // 将图片数组转换成亮度值数组，亮度值的表现形式是图片中白色像素的个数
        whitePixelCounts = symbolImages.map { self.countWhitePixelsInImage($0) },
        // 将字符数组通过亮度值就行排序
        sortedSymbols    = sortByIntensity(symbols, whitePixelCounts)
        return sortedSymbols
    }
```

其中， `sortByIntensity` 这个排序方法如下：

```swift
    private func sortByIntensity(symbols: [String], _ whitePixelCounts: [Int]) -> [String]
    {
        let
        // 用字典建立 白色像素数目 和 字符 之间的关系
        mappings      = NSDictionary(objects: symbols, forKeys: whitePixelCounts),
        // 白色像素数目数组去重
        uniqueCounts  = Set(whitePixelCounts),
        // 白色像素数目数组排序
        sortedCounts  = sorted(uniqueCounts),
        // 利用前面的字典映射，将排序后的白色像素数目转换成对应的字符，从而输出有序数组
        sortedSymbols = sortedCounts.map { mappings[$0] as! String }
        return sortedSymbols
    }
```

## 小结

简单了过了一下项目，可以隐约感觉到一些函数式风格的气息，主要体现在一下几个方面：

- `map` `reduce` 等函数的应用恰到好处，自如处理数组的转换和拼接。
- 通过 `input` 和 `output` 进行数据处理，比如 `sortByIntensity` 方法和 `symbolFromIntensity` 方法。
- 很少有状态和属性，更多的是直接的函数转换，函数逻辑不依赖外部变量，只依赖于传入的参数

代码感觉简单轻快。通过这个简单的小例子，验证了前面在 [函数式的特性](http://blog.callmewhy.com/2015/05/11/functional-reactive-programming-1/#函数式的特性) 中学习到的东西。

感觉很赞！



---

参考文献：

- [Luma Coding in Video Systems](https://en.wikipedia.org/wiki/Grayscale#Luma_coding_in_video_systems)
- [Creating ASCII art in functional Swift](http://ijoshsmith.com/2015/04/29/creating-ascii-art-in-functional-swift/)
