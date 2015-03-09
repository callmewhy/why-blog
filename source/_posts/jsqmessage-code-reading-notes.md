title: JSQMessage 源码对比阅读笔记
tags: ChatView
date: 2015-03-02 11:12:15
categories: 开发笔记
description: 同志，聊天页面你要是再这么写我就要报警了
---

[JSQMessagesViewController](https://github.com/jessesquires/JSQMessagesViewController) 是 [Jesse Squires](http://www.jessesquires.com/) 开发的一个消息界面的 UI 库。

`HBTalkTableView` 是公司自己人写的。 `HB` 的前缀实在是不知道有什么缘由，暂且理解成 哈利(Hali)波特(Bote) 的缩写吧。

一起对照阅读了两边的代码，写篇博客记录一下。

<!-- more -->

方便起见，后面 `JSQMessagesViewController` 简称为 `JSQ` ，`HBTalkTableView` 简称为 `HB` 。

## Round 1 - Overview

### HB

HB 的消息界面是这样的：

![](http://callmewhy.qiniudn.com/Screen%20Shot%202015-03-02%20at%2011.20.54.png)

### JSQ

JSQ 的消息界面是这样的：

![](https://raw.githubusercontent.com/jessesquires/JSQMessagesViewController/develop/Screenshots/screenshot0.png)

可以看到一些基本功能相同，文字、图片、视屏、电话和网址识别、长按操作、等等。

OK，旗鼓相当，第一回合双方持平。


## Round 2 - Cell

### HB

在 `HB` 里面，继承了 `UITableViewCell` 的 `HBTalkTableViewCell` 是所有 `Cell` 的基类，定义了时间戳、气泡框这些基础 view ，并且定义 `HBTalkTableCellDelegate` ，用来处理所有委托事件。

其中有个 `cellWithMessage` 的构造函数，根据传入的 `HBTalkData` 类型不同，分别 `init` ：

    + (instancetype)cellWithMessage:(HBTalkData *)message previousMessage:(HBTalkData *)previousMessage inTableView:(UITableView *)tableView previewMode:(BOOL)previewMode {
        Class cellClass = nil;
        switch (message.contentType) {
            case HBTalkDataContentTypeText:
                cellClass = (message.fromSelf)?[HBTalkTableViewTextRightCell class]:[HBTalkTableViewTextLeftCell class];
                break;
            ...
        }

        NSString *cellIdentifier = NSStringFromClass(cellClass);
        HBTalkTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
        if (cell == nil) {
            cell = [[cellClass alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier previewMode:previewMode];
        }
        cell.previousMessage = previousMessage;
        cell.message = message;
        [cell updateViews];
        return cell;
    }

然后有 `HBTalkTableViewLeftCell` 和 `HBTalkTableViewRightCell` 继承这个基类，用于定制左右两边的气泡，然后各个子类再分别继承上面的两个 `Cell` ，实现 `updateViews` 方法，完成不同类型的内容加载。

大概的结构是这样的：

![](http://callmewhy.qiniudn.com/QQ20150302-3.png)

也就是说，每当需要增加一种新类型的 `Cell` 的时候，比如我要加个发红包的 `Cell` ，那我要写两个类：`HBTalkTableViewMoneyRightCell` 和 `HBTalkTableViewMoneyLeftCell` ，分别继承自 `LeftCell` 和 `RightCell`。

看了一下现有类的实现，比如 `HBTalkTableViewTextLeftCell` 和 `HBTalkTableViewTextRightCell` ，除掉业务部分的代码，里面的属性、方法基本完全相同，唯一不同的就是气泡和消息内容的位置，存在大量重复代码。

### JSQ

再看下 `JSQ` 的源码，由于它的会话页是通过 `UICollectionView` 实现的，所以它的基类是继承了 `UICollectionViewCell` 的 `JSQMessagesCollectionViewCell` 。

对于左右气泡的问题， `JSQ` 和 `HB` 处理基本相同，都是分别建了 `JSQMessagesCollectionViewCellIncoming` 和 `JSQMessagesCollectionViewCellOutgoing` 这两个子类。 `JSQ` 是在 `xib` 里面设置不同的布局：

![](http://callmewhy.qiniudn.com/QQ20150302-4.png)

但是对于具体功能的 `Cell` ， `JSQ` 的实现方法和 `HB` 大相径庭。`JSQ` 只有 `JSQMessagesCollectionViewCellIncoming` 和 `JSQMessagesCollectionViewCellOutgoing` 这两个子类，具体业务的实现是这样做的：

- 如果是文本消息，则显示在 `textView` 上
- 如果是多媒体消息，则显示在 `mediaView` 上

然后在 `collectionView:cellForItemAtIndexPath:` 这个方法里，对数据源类型进行判断。如果是多媒体资源 (对应协议 `JSQMessageMediaData` ) ，则把多媒体资源的 `mediaView` 赋给 `cell` 的 `mediaView` ：

    // 如果是文字消息
    if (!isMediaMessage) {
        cell.textView.text = [messageItem text];
        ...
    }

    // 如果是多媒体消息
    else {
        id<JSQMessageMediaData> messageMedia = [messageItem media];
        cell.mediaView = [messageMedia mediaView] ?: [messageMedia mediaPlaceholderView];
    }

再看一下 `JSQMessageMediaData` 这个多媒体资源的协议：

    @protocol JSQMessageMediaData <NSObject>

    @required
    - (UIView *)mediaView;
    - (CGSize)mediaViewDisplaySize;
    - (UIView *)mediaPlaceholderView;
    - (NSUInteger)hash;

    @end

有了这些关键数据， `UICollectionView` 就知道应该如何正确的显示多媒体资源了。

在 `JSQ` 里提供了一个实现了 `JSQMessageMediaData` 协议的类： `JSQMediaItem` 。所有的多媒体资源类，例如图片 (`JSQPhotoMediaItem`) 、视屏 (`JSQVideoMediaItem`) 等等，都是继承自 `JSQMediaItem` 。如果要加新类型的多媒体消息，只需要自定义一个继承了 `JSQMediaItem` 的子类就可以了。

嗯拖拖拽拽个关系图是这样的：

![](http://callmewhy.qiniudn.com/QQ20150302-7.png)

我们以 `JSQPhotoMediaItem` 为例，看下图片消息的 `mediaView` ：

    ...

    #pragma mark - JSQMessageMediaData protocol

    - (UIView *)mediaView
    {
        if (self.image == nil) {
            return nil;
        }
        
        if (self.cachedImageView == nil) {
            CGSize size = [self mediaViewDisplaySize];
            UIImageView *imageView = [[UIImageView alloc] initWithImage:self.image];
            imageView.frame = CGRectMake(0.0f, 0.0f, size.width, size.height);
            imageView.contentMode = UIViewContentModeScaleAspectFill;
            imageView.clipsToBounds = YES;
            [JSQMessagesMediaViewBubbleImageMasker applyBubbleImageMaskToMediaView:imageView isOutgoing:self.appliesMediaViewMaskAsOutgoing];
            self.cachedImageView = imageView;
        }
        
        return self.cachedImageView;
    }

## Round 3 - Layout

### HB

`HB` 是基于 `UITableView` 实现的，消息时间、气泡图片都是 `Cell` 的 `subview` 。实现自适应高度 `Cell` 的方法比较淳朴，它有个静态方法计算 `Cell 的高度：

    + (float)getHeightByContent:(HBTalkData*)data preData:(HBTalkData *)preData {

        if (data.contentType == HBTalkDataContentTypeImage) {
            ...
        }
        else if (data.contentType == HBTalkDataContentTypeCall) {
            ...
        }
        else if (data.contentType == HBTalkDataContentTypeBubbleReply) {
            return 105 + height;
        }
        else if (data.contentType == HBTalkDataContentTypeVideo) {
            return 120+60;
        }
        else if (data.contentType == HBTalkDataContentTypeVoice) {
            return 42.0f + height;
        }
        else if (data.contentType == HBTalkDataContentTypePoke) {
            ...
            return height;
        }
        else if (data.contentType == HBTalkDataContentTypeGame) {
            ...
            return height;
        }
        else if ([data.contents isKindOfClass:[NSString class]]) {
            if (data.contentType == HBTalkDataContentTypeSys) {
                return 40 + height;
            }
            else if (data.contentType == HBTalkDataContentTypeGif) {
                return 130 + height;
            }
            else if (data.contentType == HBTalkDataContentTypeRegister) {
                ...
                return MAX(size.height + 14.0f, 46.0f) + 20.0f + height;
            }
            
            // 普通文字消息
            else {
                ...
                return size.height + 20 + height;
            }
        }
        
        return 0;
    }


可以看到各种 `Magic Number` 乱入，什么 130、105、120、42，就不说什么了。

如果要调整 `Cell` 的大小，需要先去 `Cell` 里调一下控件的大小，然后再调整一下这个方法里返回的高度。

然后对于左右气泡，则是在 `layoutSubviews` 里写的。比如左边气泡距离左边缘 `margin` 为10：

    - (void)layoutSubviews {
        [super layoutSubviews];
        CGRect frame= self.backgroundImageView.frame;
        frame.origin.x = 10.0f;
        self.backgroundImageView.frame = frame;
    }


### JSQ


`JSQ` 是基于 `UICollectionView` 开发的，布局任务基本全交给了 `JSQMessagesCollectionViewFlowLayout` 这个自定义的 `UICollectionViewFlowLayout` 来做，然后把布局细节通过委托方法暴露给外面。比如通过 `heightForCellTopLabelAtIndexPath` 设置时间戳的高度，如果我们希望每隔三条消息就显示一次时间戳，可以这样做：

    - (CGFloat)collectionView:(JSQMessagesCollectionView *)collectionView layout:(JSQMessagesCollectionViewFlowLayout *)collectionViewLayout heightForCellTopLabelAtIndexPath:(NSIndexPath *)indexPath {
        if (indexPath.item % 3 == 0) {
            return kJSQMessagesCollectionViewCellLabelHeightDefault;
        }
        return 0.0f;
    }

至于每个 `Cell` 的高度，则是通过 `sizeForItemAtIndexPath` 方法计算的：

    - (CGSize)sizeForItemAtIndexPath:(NSIndexPath *)indexPath {
        CGSize messageBubbleSize = [self messageBubbleSizeForItemAtIndexPath:indexPath];
        JSQMessagesCollectionViewLayoutAttributes *attributes = (JSQMessagesCollectionViewLayoutAttributes *)[self layoutAttributesForItemAtIndexPath:indexPath];
        
        CGFloat finalHeight = messageBubbleSize.height;
        finalHeight += attributes.cellTopLabelHeight;
        finalHeight += attributes.messageBubbleTopLabelHeight;
        finalHeight += attributes.cellBottomLabelHeight;
        
        return CGSizeMake(self.itemWidth, ceilf(finalHeight));
    }

先获取单个单元格的尺寸，然后加上时间戳的高度、加上用户昵称的高度、加上底部文字的高度，得出最终的尺寸。


## 小结

相比较而言， `HB` 虽然勉强实现了会话页面，但是维护成本较高，可扩展性不强，相比较而言 `JSQ` 的源码要优秀很多。[源码](https://github.com/jessesquires/JSQMessagesViewController) 中还有很多值得学习的地方，大家可以继续深入阅读。

去接女朋友了。噢耶。