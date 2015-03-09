title: 要你命三千：老代码中的那些坑
tags: WTF
date: 2015-01-20 17:55:55
categories: 开发笔记
description: 系列节目，精彩依旧，未完待续。。。
---

最近在给以前的老项目维护，说起来工作很简单，一个字：改Bug。这看起来平淡无常的工作，实际上凶险无比，藏坑无数。时至今日，感觉整个人都得到了升华。在睡觉前抽空写篇博客，和各位分享一下踩坑经历，一起品味其中的种种酸苦辣 (没甜)。

为保证个码隐私，文中代码均为化名，还望谅解。如有雷同，纯属巧合 (可以通过 `git blame` 查看是谁写的)。

<!-- more -->

## 第一回：变量命名没点数，有时写着还手误

如果要折磨一个强迫症，最好的方法就是用各种恶心的变量名恶心死他。

什么？你说首字母要大写？

    @property (nonatomic, assign) PERSONTYPE personType;


什么？你说单词里面要小写？

    typedef enum tagPersonType
    {
        person_type = 1,
        group_type,
    } PERSONTYPE;

什么？你说要用英文单词命名？

    - (void)uploadSeccess:(MessageEntity *)message;

什么？你说类前面要加前缀避免冲突？

    @interface PMWLogger : NSObject
    ...
    @interface PMTool : NSObject
    ...
    @interface MainControler : NSObject

什么？你说文件要按照目录存放？

    - Classes
        - MainControllers
            - MyController
            - Controllers
            - SettingControllers
            - ChatModel.h
            - ChatModel.m
            - SettingControllers (不是手误)
        - Chatting
        - SearchView.h
        - SearchView.m
        - Voice
        - AgentModels
    - Public 
        - Common
        - PublicDef.h
        - PublicDef.m

什么？听说OC可以用宏定义？

    #define STRHASSBUSTR(str,subStr) ...

各位看官，这，能忍？

正所谓：

> 命名拼写看心情，文件目录不分明。
  随机掺杂宏定义，鸡不安也犬不宁。


## 第二回：界面全靠神奇数，保准看到就发怵

前阵子在做 iPhone4 和 iPhone6 以及 iPhone6 Plus 的适配工作。

由于历史原因没有用 AutoLayout ，也由于历史原因老代码的布局全是用数字一个一个写死的。这就给适配带来了莫大的困难。

随便拣点代码给大家欣赏欣赏：

    UILabel *infoLabel = [[UILabel alloc] initWithFrame:CGRectMake(0, 241, 320, 28)];

0 这种数字还好说，241 就完全让人摸不着头脑，至于 320 这个改成屏幕宽度倒也就还好，但是 28 这种神奇数字又是什么呢？

这种代码就是冲着干死队友的不偿命的态度去的。虽然写起来容易，但是维护困难，可读性极差，尤其是有多个控件布局的时候，依赖关系不明显，如果调整布局需要挨个重新计算并设置值，维护起来的酸爽，谁用谁知道。

要说神奇数字，集大成者莫过于此：

    CGRect rect = CGRectMake(12.2+(page-1)*320+42.5*(i%7),((totalRows-1)%3)*55+2,42.5,42.5);

那天早上看到这代码差点就抱着键盘委屈的哭了出来。

正所谓：

> 界面写法各不同，歪门邪道千万种。
  有朝一日被辞了，你的代码我不懂。


## 第三回：私有公有混一处，代理委托亦糊涂

在聊天的时候有这样一个数据类：

    @interface HBTalkData : NSObject
    {
        UIImage *_firstImage;
        NSArray *_imageArry;
        id _contents;
    }
    @property (nonatomic, assign) NSInteger messageId;
    @property (nonatomic, strong) id contents;
    @property (nonatomic, assign) NSTimeInterval timeInterval;
    @property (nonatomic) BOOL fromSelf;
    @property (nonatomic) BOOL isGroup;
    @property (nonatomic, assign) HBTalkDataStatus talkDataStatus;
    @property (nonatomic) HBTalkDataContentType contentType;
    @property (nonatomic, strong) PersonInfo *personInfo;
    @property (nonatomic, strong) UserInfo *cardUser;
    @property (nonatomic, assign) CallType callType;
    @property (nonatomic, strong) NSString *duartion;
    @property (nonatomic, strong) NSString *mPhoneNumber;
    @property (nonatomic, strong) NSString *imageList;
    @property (nonatomic, strong) NSString *msgDesc;
    @property (nonatomic, readonly) UIImage *firstImage;
    @property (nonatomic, readonly) NSArray *imageArry;
    @property (nonatomic, assign) float     cellHeight;
    @property (nonatomic, assign) CGSize    textSize;
    @property (nonatomic) NSTimeInterval voiceDuration;
    @property (nonatomic) CGFloat dataSize;
    @property (nonatomic) NSUInteger bubbleCount;
    @property (nonatomic, copy) NSString *chatUserName;
    @property (nonatomic, strong) MessageEntity *originalMessage;
    @property (nonatomic, strong) HBTalkDataRegisterInfo *registerInfo;

    -(void)reset;
    - (NSString *)bubbleDescription;
    ...
    @end

纤弱的头文件里塞满了各种属性定义和方法定义，仿佛可以听到头文件的不满和娇喘。

给大家出个题：看下下面的内容，猜一下这个类的文件名是什么：

    ... // 此处省略20行

    @interface PersonInfo : NSObject
    ... // 此处省略20行
    @property (nonatomic, assign)BOOL     isGrey;
    @property (nonatomic, assign)BOOL     isBlack;
    @property (nonatomic, assign)BOOL     isTop;
    @property (nonatomic, assign)BOOL     isStar;

    - (BOOL)isStranger;
    - (BOOL)isIndividual;
    - (BOOL)isDuDuSecretary;

    @end

    @interface UserInfo : PersonInfo
    ... // 此处再省20行
    @property (nonatomic, assign)BOOL     mobileVerified;
    @property (nonatomic, strong)NSString *countryCode;
    @property (nonatomic, readonly)NSString *dialogName;
    @end

    @interface GroupInfo : PersonInfo
    ... // 此处又省20行
    @property (nonatomic, strong)NSString *creater;
    @property (nonatomic, assign)int      memberCount;
    @property (nonatomic, strong)NSString *members;
    @end

嗯然后这个文件叫做 `UserInfo.h` ，头文件将近100行。大兄弟，我读书少，你不要骗我。把三个类塞在一个文件里这种行为，除了难为队友，实在是没看出来有什么其他动机可言。


正所谓：

> 头文件里地方小，塞到一处并不好。
  外部对象都知道，安全问题可不小。


## 第四回：消息通知满天飞，委托方法一大堆

我一直在想，到底是什么，让这个项目的开发人员对 `NSNotificationCenter` 如此痴迷，痴迷的令人陶醉。

在通过 `Model` 调用业务逻辑的时候，它这样发了一条命令：

    // 喂，LOGIN_MODEL，帮我查下有没有更新
    [LOGIN_MODEL versionCheckFromAbout:YES];

这个业务是用 GCD 开了新线程来做的，在后台检查有没有更新，如果有更新那么版本号后面会加个感叹号。那么问题来了：你咋告诉我你检查的结果是有更新还是没更新呐？难道要写个委托？然后定义个方法？然后更新的时候指认委托？然后有了结果再告诉委托？听起来就很烦躁嘛那干脆就用通知好了：

    if (self.versionStatus != VersionStatusNormal) {
        [[NSNotificationCenter defaultCenter] postNotificationName:NOTIFY_HAS_NEW_VERSION object:nil];
    }

然后在需要做处理的类里面添加 `Observer` 就可以了：

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(myIconShouldChange) name:NOTIFY_HAS_NEW_VERSION object:nil];

哈哈哈哈搞定了。

哈哈哈哈你个头啊！整个项目里类似于这种的通知就有十来个，这还是有宏定义的，好追杀一点。对于那些没有宏定义的，随手一写复制粘贴的，不知道还要填坑多少。

通知虽好，但也不要贪杯啊。

看起来轻松，只是 `post` 了一下就搞定了，但是在 Debug 的时候有点麻烦。尤其是如果有多个 `Observer` ，改动的时候牵一发而动全身。如果真的是有这样使用的必要倒也罢了，但是本来一个 `block` 或者 `delegate` 就能简单清晰的解决，现在却被搞得这么繁重，实在是没有必要。

而且 `NSNotificationCenter` 的代码基本是一种变相的复制粘贴，十分的不工整。这是个人恩怨了，撇开不提。

`NSNotificationCenter` 这种只是不痛不痒的小问题，仅仅是逻辑不够优雅，关系不够清晰罢了。但是如果委托使用不当那是恶心的不行。看下这个聊天页面：

    @interface ChattingViewController () <UITableViewDataSource, UITableViewDelegate, UITextViewDelegate, ChattingActionsPanelDelegate, ChatModelDelegate, UIImagePickerControllerDelegate, UINavigationControllerDelegate, HBTalkTableViewCellDelegate, EGORefreshTableHeaderDelegate, XTImagePickerControllerDelegate, ChattingInputPanelDelegate, VoiceRecordingButtonTrashBinViewContainer, ChattingUserDetailPanelDelegate, VoiceRecordingButtonDelegate>

这是一个 **真实的故事** 。整个类将近3000行，有2000多行是委托里定义的方法，你能信？

在这三千行代码里漫步，万事都要小心。因为你不知道 `callIn` 这种方法到底是定义的私有方法，还是在委托里定义的方法。`#pragma mark` 自然也是看心情加的，说不定加错了你也不要当真。

有时候委托都删了不见影子了，但是委托里的各种方法还留在以前的类里。

没人敢动。

How to play.

正所谓：

> 异步回调用通知，委托多的令人痴。
  反正老子看不懂，不写代码光写诗。


## 第五回：第三方库无出处，随手改动无备注

相信做 iOS 的都知道 `AFNetworking` 这个网络库，在我们的项目里 `AFNetworking` 分两种，一个是别人家的 `AFNetworking` ，一个是咱们的 `AFNetworking`。对奏是这么任性。在一个300行的头文件里，在99行这样低调的位置里，静静的插上了自己的方法，还在上面认认真真的写上了准确的注释：

    /*扩展*/
    -(void)setDDCImageWithURL:(NSURL *)url
             placeholderImage:(UIImage *)placeholderImage
                      success:(void (^)(NSURLRequest *request, NSHTTPURLResponse *response, UIImage *image))success
                      failure:(void (^)(NSURLRequest *request, NSHTTPURLResponse *response, NSError *error))failure;

扩展个头啊！你加在人家的头文件里你说你是扩展，谁信？

这种改动遍地都是，特点是极其低调，难以察觉，甚至 `TTTAttributedLabel` 这种 UI 库也不能避免：改了 `init` 为了统一字体和颜色。。。

你说这代码，谁敢改？

我还曾经单纯的想给项目加上 `Cocoapods` 更新一下第三方库，现在想想，Naive。等以后写到新的独立模块的时候再说吧。

正所谓：

> 项目勤用三方库，随意穿插改无数。
  即使类库有更新，试问代码谁维护。


## 第六回：单个对象多职责，悲伤逆向流成河

在聊天模块有这样一个类：`ChatModel` ，简直就是个多面手。

上能和服务器聊天，上传聊天消息同步聊天记录：

    - (void)reSendMessages;
    - (void)receiveSecretaryMessage:(MessageEntity *)msgEntity;
    - (void)deleteMessagesByUserInfo:(UserInfo *)user;
    - (void)setAudioMessageBePlayed:(AudioMessageEntity *)audioMessage;
    - (void)sendBubbleReplyWithCallMessage:(CallMessageEntity *)callMessage;
    - (int)saveMessage:(MessageEntity *)message;

下能做本地缓存管理，增删改查样样精通：

    - (void)saveCacheMsg:(NSString *)msg UserMd5:(NSString *)md5;
    - (NSString *)loadCacheMsgWithMd5:(NSString *)md5;
    - (void)clearCacheMsgWithMd5:(NSString *)md5;

至于什么弹窗提醒，上传进度，完成提示，亦是轻松拿下。

以至于你改着改着不知不觉都会走到这里，因为它处理了太多太多的业务逻辑，每次 DEBUG 追杀断点回到这里，都像是一场久别重逢时的相遇，似曾相识。

正所谓：

> 一人做事一人当，切忌都往类里装。
  开发人员干的爽，维护人员很受伤。



## 第七回：产品突增新功能，一行代码变大神

有时候需求来也匆匆去也匆匆，让人猝不及防。比如一个简单的登录逻辑：

    @interface LoginModel ()
    @property (nonatomic, strong)NSString *tcpURL;
    @property (nonatomic, strong)UserInfo *offlineCallUser;
    @property (nonatomic, assign)VersionStatus versionStatus;
    @end

突然发现需要在版本更新的时候多个 API 检查，简单，加个 `BOOL` ，需要的时候设置成 YES 就行：

    @property (nonatomic, assign)BOOL isShowVersionUpdate;

但是这个功能在 `About` 页面又有点改动，简单，再加个 `BOOL` 就行：

    @property (nonatomic, assign)BOOL checkVersionFromAbout;

然后如果已经显示了注册页面又要少一些请求，行，那再加个 `BOOL` 值：

    @property (nonatomic, assign)BOOL isRegisterShow;

得了，这代码只有你能懂了：

    @interface LoginModel ()
    @property (nonatomic, strong)NSString *tcpURL;
    @property (nonatomic, strong)UserInfo *offlineCallUser;
    @property (nonatomic, assign)VersionStatus versionStatus;
    @property (nonatomic, assign)BOOL isShowVersionUpdate;
    @property (nonatomic, assign)BOOL checkVersionFromAbout;
    @property (nonatomic, assign)BOOL isRegisterShow;
    @end

想象一下实现方法里各种对 `BOOL` 标记的特殊处理，想象一下 N 个 `if` 嵌套的壮观场景。

心塞。



正所谓：

> 凡是都要听产品，各种业务催的紧。
  天塌下来也别怕，逻辑清晰自然挺。


## 第八回：来了任务有委托，多写一行都嫌多

所谓悲哀就是，当程序员发现一个 `delegate` 就能访问上级的对象，于是便把各种需要通知上级的事情都放在了委托方法里，尽管这些事情与委托本身无关，但是为了实现功能已经不在意这些所谓的设计与美观。

一个简单的 `@optional` ，甚至可以用同一个 `@protocol` 获取到各种不同的上级对象，只需要每次调用的时候加个 `respondsToSelector` 就行了。写上十几个可选方法，取一个通俗的委托名，比如 `MyDelegate` ，然后如果你持有了我但是我还想调用你的方法， so easy ，把你的方法扔到 `MyDelegate` 即可。

此时的代码便已经不再是一件艺术品，而只是一个平凡普通、毫无生机的花瓶了。


## 小结

原本还是挺欢快的吐槽，突然就不想写了。

看着以前的人写的代码，不禁有些凄凉。

在项目里用尽了各种低级下流的手段，只为了实现自己的业务。

这是对艺术的侮辱。