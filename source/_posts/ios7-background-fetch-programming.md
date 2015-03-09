title: iOS7中的后台传输服务：如何在后台获取数据
tags: iOS7
date: 2014-08-31 22:56:06
categories: 翻译笔记
description: 看完这篇文章之后我立即把后台数据关了，流量杀手。
---

在AppStore成千上万的应用中，绝大部分都需要联网进行数据交互。随着3G和4G服务的推广，用户可以保持不间断的网络连接而且不用消耗太高成本，这也就导致了联网应用数量的激增。新闻，天气，社交应用，这些都是典型的例子。在iOS7出来之前，这类应用有一个共同的缺点，就是每次加载应用的时候，用户都需要等待应用从服务器获取最新的数据。开发人员没有什么办法解决这个问题，所以他们不得不通过一些变通的方法，下载尽可能少的数据并且将控制权尽快的还给用户。

但是，随着iOS7的发布，事情有了很大的改观，这一切都得益于Background Fetch API，这是最新版本的iOS支持的多任务特性。

当后台可以获取数据的时候，系统就会在后台不时的唤醒应用，获取最新的数据刷新页面。通过这种方式，应用的数据总是保持最新的状态，用户在启动应用的时候不用等待加载的时间。换句话说，这是一个所有开发者期待很久的一个工具，让应用刷新数据同时还不会让用户不愉快。在这篇教程里，我们将会看看这个新特性是如何加入到应用中的。不过，为了能充分的理解Background Fetch，我们先来看看另一个重要的细节。
![](http://www.appcoda.com/wp-content/uploads/2014/03/background-app-refresh.jpg)

在应用中使用Background Fetch API很容易，只需要遵循以下三步：

- 在应用的Multitasking Capabilities开启
- 在`application:didFinishLaunchingWithOptions: `委托方法里设置系统在后台获取数据的时间间隔
- 在AppDelegate里实现新的方法，用来处理获取数据的结果。

在这里解释一下Background Fetch的用途。它并不是只用来处理联网数据，也可以用来执行应用中的任务。不过这是很少见的情况，大多数情况下都是用来解决后台获取数据的问题。

在后面的内容中，我们将在一个示例项目中看到上面三个步骤的实现细节。现在，我想多说一点关于第二步的内容，让大家更便于理解。先解释一个名词：抓取间隔(fetching interval)，意思是系统唤醒应用并允许它加载数据的时间间隔。有两种方式可以设置抓取间隔的值：使用系统预定义的值，或者自定义(NSTimeInterval)。不过我们需要考虑一些内容：后台抓取的约频繁，占用的资源就越多。iOS有一套自我保护机制，会限制访问API过于频繁的应用，所以在设置自定义间隔的时候一定要谨慎。使用iOS提供的默认值可能会是最好的选择。不幸的是，后台获取数据会导致电池消耗的非常快。

Background Fetch有一个很酷炫的特性，它可以通过学习知道应该让应用在后台启动的时间。假设有一个用户每天早上八点半运行应用(伴着热咖啡阅读最新的新闻资讯)，在用过几次之后，系统发现用户很有可能下次在同样的时间再次运行应用，所以它很小心的在通常启动时间之间启动了应用并获取了数据(比如在早上八点)。这样当用户再次打开应用的时候，最新的内容就会被及时呈现出来，乖乖的等待阅览，不用等待加载的时间。这个特性被称之为使用预测(usage prediction)。不过要注意，只有在你让系统决定抓取间隔的时候才会有这个特性。

当使用Background Fetch的时候，你需要注意一些约束。分配给应用的后台执行的时间间隔并不是无限的，iOS提供一个30秒的边框让应用被唤醒，抓取数据，刷新UI，然后重新回到睡眠状态。你需要确保你的每个执行的任务都会在对应的30秒内完成，要不然系统就会中断它们。如果30秒确实不够用，可以调用后台传输服务接口(Background Transfer Service API)，我们将在未来的某篇教程中讲解。所以，时刻谨记并且多多思考，要不然你的应用可能会遭遇意外。

在前言结束之前，我还想再说一些东西。首先，Background Fetch在程序正在运行的状况下也是有效的。在这种情况下，它只会刷新内容。其次，Background Fetch API应该用于一些不重要的数据更新，因为即使设置了相关参数，系统还是有可能没能在后台启动程序。

所以，在说了那么多内容之后，现在是时间开始动手学习啦。在阅读教程的时候，注意在这个教程中我们将要做什么，以及在下一步我们将要做什么，以便充分的理解并应用Background Fetch API。


## 应用预览

为了充分了解Background Fetch API，最理想的实践方案开发一款利用了后台更新技术的新闻客户端，而这就是我们接下来要做的。我们的示例应用将会非常简单，用来演示Background Fetch API的最佳使用方式。不过在一开始，让我们先来大概的了解一下。

在我们的应用中，我们将会从[Reuters](http://www.reuters.com/tools/rss)的RSS订阅中抓取最新的标题以及发布日期和链接。我们不会再抓取其他数据，因为两条原因：1.我们不想超出30秒的时间范围(在极慢的3G网络下这是有可能的)。2.尽可能的保持应用简单，便于大家理解。所以，我们将会选择一个订阅数据，将它解析并在TableView中展示。假想用户在这个应用中可以手动通过`RefreshControl`刷新。当所有都准备好了，功能基本完善的时候，我们将会加入Background Fetch API部分，并且我们将会测试它能否在后台获取数据。在虚拟机上有特殊的实现手段，在后面我们将会说到。

考虑到应用的结构，我们基于`Single View Application`模板创建一个新应用。正如我前面所说，我们将会添加一个`UITableViewController`以及一个`RefreshControl`以供用户手动刷新数据。下载下来的数据将会被存储到NSArray里并且数组中的每个对象都是NSDictionary，存储了新闻的标题，发布时间以及对应的链接地址。在成功更新数据之后，数组的内容将会被永久的写入到本地目录。所以当我们使用Background Fetch的时候会和本地文件进行比较，从而得知是否有最新的内容。除此之外，我们还会加一个toolbar，上面有一个按钮，允许我们删除数据文件重新开始。

为了充分利用Reuters的RSS订阅功能，当点击新闻标题的时候，我们将通过Safari打开并加载完整的文章。这样，每个文章对应的链接地址就都派上了用场。
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_1_app_sample.jpg)


## 初步应用
现在需要迫切解决的一个问题是，下载下来的数据会是XML格式，而我们需要一个XML的格式解析工具，将需要的数据抽离出来。为了方便起见，你可以在这里下载[初步项目](https://dl.dropboxusercontent.com/u/2857188/BackgroundFetchStarterProject.zip)，在这个项目里你会看到一个`XMLParser`用来将XML格式的数据抽离出来。当然这只是一个工具而已，应用的其他部分还未完成，万事俱备，只需要按照教程一步一步的走下去就可以了。

在继续之前，先看一下下载的XML格式的示例数据：
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_2_xml_sample.jpg)


## 设置界面

让我们开始用Interface Builder构建页面。选中Main.storyboard文件，打开Utilities面板，从Objects Library拖拽一个UITableView和一个Toolbar到画布上。设置属性如下：

- UITableView
Frame: X=0, Y=20, Width=320, Height=504
- Toolbar
Frame: X=0, Y=524, Width=320, Height=44
- Bar Button
Identifier: Trash


当你完成了上面的工作，拖拽一个UITableViewCell到TableView上。选中它并且在Attributes Inspector面板将Style设置为Subtile，将Identifier设置为idCellNewsTitle。
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_3_cell_setup.jpg)

最后，点击Cell里的Title标签，在Attributes Inspector面板设置字体为系统字体，粗体，15号字体。然后将Lines设置为3，你的场景看起来会是这样的：
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_4_ib_scene.jpg)

现在我们已经添加了必要的页面元素，接下来我们可以创建IBOutlet属性连接UITableView，创建IBAction方法连接到删除(Trash)按钮。

点击ViewController.h文件添加如下内容：
```
@interface ViewController : UIViewController
@property (weak, nonatomic) IBOutlet UITableView *tblNews;
- (IBAction)removeDataFile:(id)sender;
@end
```

在回到页面编辑之前，让我们先给我们的类加上UITableViewDelegate和UITableViewDatasource这两个委托。修改@interface：
```
@interface ViewController : UIViewController <UITableViewDelegate, UITableViewDataSource>
```

很好，回到Storyboard文件，在左侧文档面板中的View Controller上按住control点击鼠标左键，或者点击鼠标右键。在黑色弹出窗口中点击tblNews左侧的小圆点拖拽到右边的TableView上。IBOutlet连接成功！按照同样的步骤将IBAction连接到Trash按钮上。
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_5_iboutlet_connect.jpg)

## UIRefreshControl和一些代码层面的配置
正如你说看到的，界面的设置十分容易，而且我们只需要连接一个IBOutlet和一个IBAction方法。现在，我们应该把重心放在代码上了。一开始我们需要做两件事情：创建一个RefreshControl让我们可以下拉刷新我们的数据，以及声明并初始化一些我们需要的对象。

打开ViewController.m文件，进入到类的私有部分，声明如下内容：
```
@interface ViewController ()
@property (nonatomic, strong) UIRefreshControl *refreshControl;
@property (nonatomic, strong) NSArray *arrNewsData;
@property (nonatomic, strong) NSString *dataFilePath;
@end
```

简单的解释一下：

- refreshControl：不用说太多，在刷新TableView的数据的时候这个会显示在UITableView的上面。
- arrNewsData：这个数组包含将在TableView中展示的真实数据，正如我们将在后面看到的，这个数组存储的内容是NSDictionary，包含三个值：标题，发布时间，链接。
- dataFilePath：存储下载的数据的文件路径。

接下来让我们看下`viewDidLoad`方法，在这里我们可以做一些初始化。在下面的代码片段里，我们需要完成如下任务：

- 我们将会把self设置为TableView的delegate和datasource
- 我们将指定存储文件的路径并且将它存在dataFilePath属性里
- 我们将会初始化refreshControl

让我们来看看下面的代码：
```
- (void)viewDidLoad
{
    [super viewDidLoad];
    // 1. Make self the delegate and datasource of the table view.
    [self.tblNews setDelegate:self];
    [self.tblNews setDataSource:self];
    // 2. Specify the data storage file path.
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *docDirectory = [paths objectAtIndex:0];
    self.dataFilePath = [docDirectory stringByAppendingPathComponent:@"newsdata"];
    // 3. Initialize the RefreshControl.
    self.refreshControl = [[UIRefreshControl alloc] init];
    [self.refreshControl addTarget:self
                            action:@selector(refreshData)
                  forControlEvents:UIControlEventValueChanged]; 
    [self.tblNews addSubview:self.refreshControl];
}
```

我认为这个代码足够简单并且很容易理解，关于RefreshControl说两条：第一：selector中的refreshData方法是一个私有方法，在稍后我们将会声明，这个方法将会在每次刷新的时候调用。第二：使用`[self.tblNews addSubview:self.refreshControl]; `我们将会把下拉刷新作为一个子视图(subview)加到TableView里面。如果你不加这行代码，那么在你尝试刷新数据的时候什么都不会发生。

注释：我们的类是ViewController的子类，TableView已经作为一个子视图(subview)加载了主视图里。在这种情况下，RefreshControl必须手动加到子视图里。但是，如果你的ViewController是UITableViewController的子类，并且RefreshControl已经是它的一个属性了，那么就不用手动添加了。

最后，在#import下面添加一行，定义数据源：
```
#define NewsFeed @"http://feeds.reuters.com/reuters/technologyNews"
```


## 刷新数据
现在我们的基础工作基本就算完成了，接下来让我们完成刷新数据的功能。关于“刷新”一词，我并不想获取任何可供下载的数据，而只是在应用完全没有数据的时候，获取初始化的数据。在前面的代码中我们在selector里用了一个名为`refreshData`的方法，而我们还没有定义这个方法，Xcode应该会因为这个原因报错。

所以，前往类的私有区域声明如下方法：
```
@interface ViewController ()
...
-(void)refreshData;
@end
```

在继续之前，因为我们会用到`XMLParser`这个自定义类，我们需要在一开始引入头文件：
```
#import "XMLParser.h"
```

很好，接下来让我们来看一下方法的实现部分。首先我们加入以下代码：
```
-(void)refreshData{
    XMLParser *xmlParser = [[XMLParser alloc] initWithXMLURLString:NewsFeed];
    [xmlParser startParsingWithCompletionHandler:^(BOOL success, NSArray *dataArray, NSError *error) {     
    }];
}
```
代码里做的第一件事情是初始化一个XMLParser对象，把NewsFeed作为参数，所以它知道去哪里获取数据。接下来我们调用`startParsingWithCompletionHandler:`方法，它将开始获取数据并将其进行格式解析，在解析结束的时候Completion handler将会被调用，这样我们就可以在block里管理下载的数据。对于block的参数做一些注释：

- success：用来表示数据是否解析成功的标志
- dataArray：存储解析后数据的数组
- error：如果发生错误，返回一个指向错误的指针


接下来要做的是我们将如何管理解析后的数据。大概有如下四个任务：

- 将解析后的数据存到我们前面声明的arrNewsData数组里。
- 重新加载TableView让它显示我们下载的数据
- 数据持久化存储
- 将下拉刷新的RefreshControl的动画停止

在这里可以提前透露一下，在我们实现Background Fetch的时候，这三个任务也将会被调用。所以我们最好把它们放在另一个私有方法里，在那个方法里执行这些任务，而不是用到一次就写一次(说实话，我真的很讨厌一遍又一遍的写着一样的代码，那种感觉不要太糟)。

在类的私有区域定义如下方法：
```
@interface ViewController ()
...
-(void)performNewFetchedDataActionsWithDataArray:(NSArray *)dataArray;
@end
```

接下来，在里面实现前面提到的三个任务：
```
-(void)performNewFetchedDataActionsWithDataArray:(NSArray *)dataArray{
    // 1. Initialize the arrNewsData array with the parsed data array.
    if (self.arrNewsData != nil) {
        self.arrNewsData = nil;
    }
    self.arrNewsData = [[NSArray alloc] initWithArray:dataArray];
    // 2. Reload the table view.
    [self.tblNews reloadData]; 
    // 3. Save the data permanently to file.
    if (![self.arrNewsData writeToFile:self.dataFilePath atomically:YES]) {
        NSLog(@"Couldn't save data.");
    }
}
```

正如你所看到的，一切都很轻松，接下来我们回到refreshData方法，在completion handler的block里调用：
```
-(void)refreshData{
    XMLParser *xmlParser = [[XMLParser alloc] initWithXMLURLString:NewsFeed];
    [xmlParser startParsingWithCompletionHandler:^(BOOL success, NSArray *dataArray, NSError *error) {     
        if (success) {            
            [self performNewFetchedDataActionsWithDataArray:dataArray];
            [self.refreshControl endRefreshing];
        }
        else{
            NSLog(@"%@", [error localizedDescription]);
        }
    }];
}
```

我们先检查一下success标志，如果为TRUE，也就是说我们的数据下载并解析成功，则调用`performNewFetchedDataActionsWithDataArray:`方法，并且用endRefreshing方法停止rfresh control的刷新动画。对于解析出错的情况，我们简单处理，将错误打印出来。

这样应用的刷新功能就算完成了，可以继续实现剩下的功能。


## 展示数据

既然我们已经可以使用RefreshControl刷新数据，接下来就是如何将数据在TableView中展示了。我们需要实现必要的UITableView的delegate和datasource方法，添加如下代码：
```
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return 1;
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return self.arrNewsData.count;
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"idCellNewsTitle"];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:@"idCellNewsTitle"];
    }
    NSDictionary *dict = [self.arrNewsData objectAtIndex:indexPath.row];
    cell.textLabel.text = [dict objectForKey:@"title"];
    cell.detailTextLabel.text = [dict objectForKey:@"pubDate"];    
    return cell;
}

-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    return 80.0;
}

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    NSDictionary *dict = [self.arrNewsData objectAtIndex:indexPath.row];
    NSString *newsLink = [dict objectForKey:@"link"]; 
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:newsLink]];
}
```

有两个地方需要解释一下。

首先是这个部分：
```
NSDictionary *dict = [self.arrNewsData objectAtIndex:indexPath.row]; 
cell.textLabel.text = [dict objectForKey:@"title"];
cell.detailTextLabel.text = [dict objectForKey:@"pubDate"];
```

前面已经提到过，arrNewsData中的每一个对象都是一个NSDictionary对象，包含三个值，标题，发布时间，和链接地址，他们的key分别是：title，pubDate，link。在上面的代码里，我们取出字典，并将其中的title对象分配给cell的TextLabel，将其中的pubDate对象分配给DetialTextLabel。


第二个需要解释的是`tableView:didSelectRowAtIndexPath: `这个方法。在这个方法里我们获取到与标题对应的链接地址，然后通过`sharedApplication`的`openUrl:`方法通过safari加载这个链接地址。


现在我们可以运行一下我们的应用，下拉TableView你会看到RefreshControl已经有用了并且在TableView里展示了获取到的新数据。


## 加载数据
在我们前面你已经看到每次加载完数据我们都会将他持久化存储到本地，所以我们在每次运行程序的时候也需要把数据再读取出来，而不是每次运行程序都空荡荡的。加载本地数据十分容易，你只需要在`viewDidLoad`方法里添加如下内容：

```
- (void)viewDidLoad
{
    ...
    if ([[NSFileManager defaultManager] fileExistsAtPath:self.dataFilePath]) {
        self.arrNewsData = [[NSMutableArray alloc] initWithContentsOfFile:self.dataFilePath];     
        [self.tblNews reloadData];
    }
}
```

我们先检查一下指定路径下的文件是否存在，如果存在则用本地数据初始化`arrNewsData`数组然后重新加载TableView的数据。

下一次在你运行应用的时候，如果数据文件已经存在，你会发现TableView已经自动填充好了数据。酷！


## 后台获取

现在我们的程序已经可以获取数据并将其在TableView中展示了，是时候开始关于Background Fetch的开发了。正如我在介绍中讲的，整个步骤分为三步完成，我们将会详细的学习每一步的内容。

我们先在Xcode里开启相关的功能。点击左侧目录中的项目文件：
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_6_project_target.jpg)

接下来，在Xcode的主面板，点击Capabilities标签，打开Background Modes的开关。在下面的选项中，点击勾选Background Fetch选项，这样Xcode就会自动添加相关的值到你项目的info.plist文件中。

![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_7_bgfetch_enable.jpg)

很好，第一步已经完成。

接下来打开AppDelegate.m文件，在`application:didFinishLaunchingWithOptions:`方法里天加一行代码，定义Background Fetch的执行频率。在我们的程序里，使用`UIApplicationBackgroundFetchIntervalMinimum`这个预定义的值，让系统去决定它具体是多少，确保最短的时间间隔。代码如下：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [application setMinimumBackgroundFetchInterval:UIApplicationBackgroundFetchIntervalMinimum]; 
    return YES;
}
```

在第三步，也就是最后一步里，我们将定义一个新的委托方法。在这个方法里我们处理Background Fetch的对应操作。事实上在这个方法里，我们必须实现所有应用抓取数据必需的业务逻辑，并且管理数据，刷新界面。这个方法里面的内容取决于每个应用，所以很难总结什么通用的规则。

让我们看下这个委托方法：
```
-(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    // We will add content here soon.
}
```

当程序在后台运行并且通过Background fetch获取最新的数据的时候`application:performFetchWithCompletionHandler:`这个委托方法就会被调用并且执行里面的所有任务。最后一步是成功完成之后的回调，回调方法需要提供一个参数`UIBackgroundFetchResult`，有以下三种结果：

- UIBackgroundFetchResultNoData：表示没有新数据
- UIBackgroundFetchResultNewData：获取到了新数据
- UIBackgroundFetchResultFailed：获取数据失败


来看一下我们的程序。当Background Fetch抓取到数据的时候，我们希望XMLParser能够从新的订阅内容里下载并且解析数据然后刷新TableView。换句话说，我们希望能够实现差不多和下拉刷新一样的操作，就像是前面RefreshControl中实现的功能一样。另外，考虑到有可能没有获取到新的数据，或者整个过程中可能会报错，我们将会用合适的参数去调用完成的回调方法。


最适合完成这些任务的地方是ViewController里面，这样我们就可以直接获取到所有的对象和控制权。所以我们离开了delegate文件，然后打开ViewController.h文件声明一个public公有方法，这个方法将会在`application:performFetchWithCompletionHandler:`这个委托方法中被调用。声明如下：
```
@interface ViewController : UIViewController <UITableViewDelegate, UITableViewDataSource>
...
-(void)fetchNewDataWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler;
@end
```

接下来在ViewController.m文件里实现这个方法。在我们这么做之前，先解释一下基本的处理逻辑：用XMLParser类下载并解析数据，然后检查一下最新的内容和本地已存在的内容是否相同(希望你此时能够理解持久化处理的作用)，根据比较的结果我们将会调用`completionHandler`方法，并且提供对应不同结果的参数，也就是上面列举的那三个结果参数。如果获取到了新的数据，我们将调用`performNewFetchedDataActionsWithDataArray:`这个私有方法，调用抓取到新的数据后会调用的对应方法。

```
-(void)fetchNewDataWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    XMLParser *xmlParser = [[XMLParser alloc] initWithXMLURLString:NewsFeed];
    [xmlParser startParsingWithCompletionHandler:^(BOOL success, NSArray *dataArray, NSError *error) {
        if (success) {
            NSDictionary *latestDataDict = [dataArray objectAtIndex:0];
            NSString *latestTitle = [latestDataDict objectForKey:@"title"];
            NSDictionary *existingDataDict = [self.arrNewsData objectAtIndex:0];
            NSString *existingTitle = [existingDataDict objectForKey:@"title"];
            if ([latestTitle isEqualToString:existingTitle]) {
                completionHandler(UIBackgroundFetchResultNoData);
                NSLog(@"No new data found.");
            }
            else{
                [self performNewFetchedDataActionsWithDataArray:dataArray];
                completionHandler(UIBackgroundFetchResultNewData);
                NSLog(@"New data was fetched.");
            }
        }
        else{
            completionHandler(UIBackgroundFetchResultFailed);
            NSLog(@"Failed to fetch new data.");
        }
    }];    
}
```

这个方法可以说是我们应用中的核心部分，注意到我们用`completionHandler`对不同结果进行处理。

所以，我们的方法会解决BackgroundFetch运行时的所有问题，让我们回到AppDelegate.m文件中实现`application:performFetchWithCompletionHandler: `方法中的内容。在这之前，先在前面引入头文件：

```
#import "ViewController.h"
```

在委托方法里，用rootViewController属性可以获取到ViewController对象，这样就可以用其中的公有方法了：

```
-(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    ViewController *viewController = (ViewController *)self.window.rootViewController; 
    [viewController fetchNewDataWithCompletionHandler:^(UIBackgroundFetchResult result) {
        completionHandler(result);    
    }];
}
```

正如你所看见的，`fetchNewDataWithCompletionHandler:`的block提供了结果状态参数，我们只需要将它传给对应的方法就可以了。

就是这样了！我们已经实现了BackgroundFetch的对应功能。


## 测试应用

在测试之前我们需要走一些准备工作。点击项目的schemes，选择ManageSchemes选项：
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_8_manage_schemes.jpg)

在弹出的窗口中，确保BackgroundFetchDeme是被选中的。然后点击左下方的齿轮按钮，在菜单中选择Duplicate选项。

![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_9_duplicate.jpg)

这时会出现一个新的窗口，你可以按照下面步骤执行：

- 设置名称为BackgroundFetchDemo_Test，或者其他你喜欢的名字。
- 在左边面板，选择Run标签。
- 在主面板，选择Options标签。
- 选中Launch due to a background fetch event。

![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_10_setup_scheme.jpg)

点击OK按钮，关闭所有窗口。
在Xcode的工具栏，选择新的scheme然后运行程序。你会注意到不会再像以前那样弹出虚拟机来运行程序，但是在debugger里你会看到BackgroundFetch执行时的输出。如果有新的数据，只需要点击虚拟机中的应用图标，然后你就会发现TableView中展示的是最新的数据！想象一下这个发生在你的应用里，你的用户们会发现一运行程序里面的数据就已经是最新的了。不可思议啊！

为了再在正常模式下运行程序，只需要选择BackgroundFetchDemo这个scheme就可以了。当然，在运行程序的时候你也可以测试这个功能，这样就不需要切换scheme了，只需要在Xcode中选择Debug > Simulate Background Fetch就可以了。

## 跟踪时间

在教程的介绍里，我提到系统处理BackgroundFetch的时候有一个30秒的限制，如果需要的时间大于30秒，系统将会有一些其他处理。不过，我们倒是很想知道我们的应用在后台获取数据的时候用了多少时间。所以我们在代码里加入一些内容，让程序在完成后台获取之后告诉我们到底花了多少时间。

前往AppDelegate.m文件，在`application:performFetchWithCompletionHandler:`委托方法里添加一些代码。我们要做的工作很简单：保存BackgroundFetch的开始和结束时间，并且计算二者之间的时间间隔。计算结果将会在debugger中打印出来。

下面是改进之后的委托方法：

```
-(void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    NSDate *fetchStart = [NSDate date];
    ViewController *viewController = (ViewController *)self.window.rootViewController;
    [viewController fetchNewDataWithCompletionHandler:^(UIBackgroundFetchResult result) {
        completionHandler(result);
        NSDate *fetchEnd = [NSDate date];
        NSTimeInterval timeElapsed = [fetchEnd timeIntervalSinceDate:fetchStart];
        NSLog(@"Background Fetch Duration: %f seconds", timeElapsed);     
    }];
}
```

添加的代码很明显，选中合适的scheme运行项目测试BackgroundFetch的功能。这次你会看到整个过程的完成时间。正如在debugger里面看到的，我们的应用完全不用担心30秒的时间限制：
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_11_debugger_duration.jpg)


## 删除数据

如果你还记得，在设计UI的时候，我们添加了一个Toolbar以及一个删除按钮，并且把它连到了一个IBAction上。这个方法是用来删除数据的，不过现在我们还没有实现这个方法。既然BackgroundFetch功能已经开发完毕，那么让我们继续完成这个`removeDataFile:`方法。

这个IBAction很简单，我们只需要检测一下文件是否存在即可，如果存在则删除数据。然后我们将数组设为nil并且刷新页面：

```
- (IBAction)removeDataFile:(id)sender {
    if ([[NSFileManager defaultManager] fileExistsAtPath:self.dataFilePath]) {
        [[NSFileManager defaultManager] removeItemAtPath:self.dataFilePath error:nil];
        self.arrNewsData = nil;     
        [self.tblNews reloadData];
    }
}
```


## 运行应用

如果你是那种喜欢先实现功能然后运行程序的人，那么现在就是你动手的时候了。如果你前面没有运行过这个应用，现在可以分别尝试一下两种scheme。最重要的是，理解BackgroundFetch。
![](http://www.appcoda.com/wp-content/uploads/2014/03/t7_12_refreshing_data.jpg)


## 总结

BackgroundFetch是我觉得iOS中最酷最实用的特性之一。你可以单独使用这个功能，也可以和其他API结合使用，你可以创建出完全改变用户体验的应用。开发应用的时候，让用户开心永远是最高目标。总的来说，iOS7带来了很多新的变化，充分利用它们，你可以开发出不可思议的绝佳应用！

你可以[点击这里](https://dl.dropboxusercontent.com/u/2857188/BackgroundFetchFinalProject.zip)下载源码。




---

原文地址：

- [Background Transfer Service in iOS 7 SDK: How To Download File in Background](http://www.appcoda.com/background-transfer-service-ios7/)