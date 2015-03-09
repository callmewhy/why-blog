title: 分分钟给你的应用做一个 Bug 上报小工具
tags: Bug
date: 2015-01-15 23:14:28
categories: 开发笔记
description: 重返阔别已久的 PHP 大法，代码写的不好大家不要嘲笑我哈哈。
---

## 提示

发现其实上报的文件压缩一下再上传可以省掉很多事情，各位可以直接跳转到最后一段：[更新](#更新)。

## 简介

前阵子在 Github 上看到一个挺有趣的小项目：[BugHunt-iOS](https://github.com/etsy/BugHunt-iOS)，可以在屏幕上召唤一个小 Bug 按钮，双击截屏，单击开始上传 Bug。动图如下：

![](https://github.com/etsy/BugHunt-iOS/raw/master/bughunt.gif)

感觉这是个挺有趣的功能，而且放在内部测试的时候可以让同事方便的上传 Bug ，便于进一步改进。

于是今天就试着用 PHP 搭建个本地服务器，试一下 Bug 上传功能。

## 客户端

具体的 BugHunt 的使用方法可以看 README 说明，看完之后我们知道，需要一个实现了 `EBHNetworkCommunicator` 协议的网络连接类来进行网络管理。示例代码中给了我们一个简单的例子：


    #import "MyNetworkCommunicator.h"

    // Models
    #import "EBHBugReport.h"
    #import "EBHTask.h"
    #import "EBHLeaderboardUser.h"

    @implementation MyNetworkCommunicator

    - (BOOL)createBugHuntIssue:(EBHBugReport *)bugReport
                      completion:(EBHCreateNewIssueCompletionBlock)completionBlock
    {
        ... 
    }

    - (BOOL)fetchBugHuntTasks:(EBHFetchTasksCompletionBlock)completionBlock
    {
       ...
    }

    - (BOOL)fetchBugHuntLeaderboard:(EBHFetchLeaderboardCompletionBlock)completionBlock
    {
        ...
    }

    @end


在使用的时候我们只需要给 BugHunt 指定对应的 `NetworkCommunicator` 即可：

    MyNetworkCommunicator *networkCommunicator = [[MyNetworkCommunicator alloc] init];
    [BugHunt setNetworkCommunicator:networkCommunicator];

在 `- (BOOL)createBugHuntIssue: completion:` 这个方法里，我们可以进行 Bug 的具体上传设置。

根据我的需求，先生成了一个文件名用来存储在服务器端，首先它要好认！

所以文件名的格式是：用户名+三位随机数+当前日期的时间戳+索引值

示例：nouser292-2015-01-15-20-36-18-0

嗯，这个示例中没有用户名，随机数是292，日期是2015年01月15日20时36分18秒，是上传的第0份文件，很好！

这样我在上传的时候只需要统一安排这样的参数名，不管有多少文件都可以上传，顺便再把文件名和文件数目传过去，这样服务器就可以根据这两个值生成文件列表，从而获取 POST 请求中的所有文件。

先看下 iOS 端的上传代码：

    - (BOOL)createBugHuntIssue:(EBHBugReport *)bugReport
                    completion:(EBHCreateNewIssueCompletionBlock)completionBlock
    {
        // 生成用户名
        NSString *username = bugReport.username;
        if (!username) {
            username = [NSString stringWithFormat:@"nouser%d",arc4random() % 1000];
        }
        
        // 生成一个临时的文件名
        NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
        [dateFormatter setDateFormat:@"yyyy-MM-dd-HH-mm-ss"];
        NSString *strDate = [dateFormatter stringFromDate:[NSDate date]];
        NSString *fileName = [NSString stringWithFormat:@"%@-%@-", username, strDate];
        
        
        NSArray *screenshots = bugReport.screenshots;

        [_manager POST:@"http://192.168.31.97:8888/index.php/bug/do_upload" parameters:@{@"count":@(screenshots.count),@"filename":fileName} constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
            for (int i = 0; i < screenshots.count; i ++) {
                UIImage *shot1 = screenshots[i];
                NSData *imageData = UIImageJPEGRepresentation(shot1, 0.5);
                NSString *imageName = [NSString stringWithFormat:@"%@%d",fileName,i];
                [formData appendPartWithFileData:imageData name:imageName fileName:imageName mimeType:@"image/jpeg"];
            }
        }
               success:^(AFHTTPRequestOperation *operation, id responseObject) {
                   NSLog(@"%@",responseObject);
                   completionBlock(nil);
               }
               failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                   completionBlock(error);
               }];
        return YES;
    }

确实挺简单的，主要就是把截屏的 `UIImage` 转换成 `NSData` 然后上传就 OK 了。接下来看下服务器端的实现。


## 服务器端

由于只是个简单的小功能，所以 PHP 端我选择了轻量级的 CodeIgniter 框架 (其实选择 CI 的原因是其他几个好久不玩基本都忘了，但是 CI 只要看着文档就能搞，基本没有学习成本)。

简单的部署好了之后，我们先添加一个 `bug_controller` 作为 Bug 上报事件的控制器。

关键的代码就这么几行：

    function do_upload()
    {
        // 获取上传的文件数目
        $count = $this->input->post('count');

        // 获取上传的文件名称
        $filename = $this->input->post('filename');

        // 初始化上传工具类
        $config['upload_path'] = './uploads/';
        $config['allowed_types'] = '*';
        $this->load->library('upload', $config);
        $this->upload->initialize($config);

        for ($x = 0; $x < $count; $x++) {
            $this->upload->do_upload($filename.strval($x));
        }

        $arr = array ('count' => $count, 'filename' => $filename.strval(1), 'error' => $this->upload->display_errors());
        $this->output->set_content_type('application/json')->set_output(json_encode($arr));

        return;


    }

不到十行的代码就完成了上传文件的处理，由于有 CodeIgniter 的工具类，所以上传只需要 `$this->upload->do_upload($filename);` 这么一行代码就可以了。

我们的主要工作就是对文件名的解析，因为上传的文件都在 `$_FILES` 这个变量里。其实也可以遍历所有的 key 值然后挨个上传，但是我当时就是秀逗了然后用了个比较简单粗暴的方法实现了 (其实根本原因就是对 PHP 不熟悉嗯)。


## 小结

目前只是完成了文件的上传，其实还剩下了一些工作要做，比如搞个数据库存储用户名、存储问题描述，再拿 BootStrap 或者 Foundation 搭个好看的页面用来展示数据等等。

不过由于当初的目的只是为了上传 Log 文件，所以就先做到这里了。等完善了 Web 端的展示再来更新。


## 更新

首先对于前面的种种繁琐，原因在于服务器不知道客户端上传了多少文件 (其实感觉用 `$_FILE` 这个字典应该能获取的但是就是懒)。其实有个更简单的处理方案：压缩之后再上传。

我们首先需要一个压缩工具对数据进行压缩，于是找到了 [Objective-Zip](https://github.com/flyingdolphinstudio/Objective-Zip) 。接下来就是修改一下我们的代码，把本来放在 `Form` 表单里的内容放到 `ZIP` 文件中，然后把压缩之后的 `ZIP` 文件上传服务器。

既然一起打包，那我们可以把问题描述写到一个文件里然后一起上传。修改之后的部分代码如下：


    - (BOOL)createBugHuntIssue:(EBHBugReport *)bugReport
                    completion:(EBHCreateNewIssueCompletionBlock)completionBlock
    {
        // 生成用户名
        NSString *username = bugReport.username;
        if (!username) {
            username = [NSString stringWithFormat:@"nouser%d",arc4random() % 1000];
        }
        
        // 生成一个临时的文件名
        NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
        [dateFormatter setDateFormat:@"yyyy-MM-dd-HH-mm-ss"];
        NSString *strDate = [dateFormatter stringFromDate:[NSDate date]];
        NSString *fileName = [NSString stringWithFormat:@"%@_%@", username, strDate];
        NSArray *screenshots = bugReport.screenshots;
        
        // 设置超时
        [_manager.requestSerializer setTimeoutInterval:10];

        [_manager POST:@"http://xxx.xxx/xxx" parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
            
            NSString *documentsDir= [NSHomeDirectory() stringByAppendingPathComponent:@"Documents"];
            NSString *filePath= [documentsDir stringByAppendingPathComponent:fileName];

            ZipFile *zipFile= [[ZipFile alloc] initWithFileName:filePath mode:ZipFileModeCreate];
           
            // 把问题描述写到文件里一起压缩
            ZipWriteStream *stream= [zipFile writeFileInZipWithName:@"debug_description.log" compressionLevel:ZipCompressionLevelBest];
            [stream writeData:[bugReport.bugDescription dataUsingEncoding:NSUTF8StringEncoding]];
            [stream finishedWriting];

            // 把缩略图放进压缩文件
            for (int i = 0; i < screenshots.count; i ++) {
                UIImage *shot1 = screenshots[i];
                NSData *imageData = UIImageJPEGRepresentation(shot1, 0.5);
                NSString *imageName = [NSString stringWithFormat:@"%@_%d.jpg",fileName,i];
                ZipWriteStream *stream= [zipFile writeFileInZipWithName:imageName compressionLevel:ZipCompressionLevelBest];
                [stream writeData:imageData];
                [stream finishedWriting];
            }
            [zipFile close];
            
            // 取出压缩文件加到表单中
            NSData *reader = [NSData dataWithContentsOfFile:filePath];
            NSString *upFileName = [NSString stringWithFormat:@"iOS_%@.zip", fileName];
            [formData appendPartWithFileData:reader name:@"bugfile" fileName:upFileName mimeType:@"application/zip"];
        }
               success:^(AFHTTPRequestOperation *operation, id responseObject) {
                   NSLog(@"%@",responseObject);
                   completionBlock(nil);
               }
               failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                   completionBlock(error);
               }];
        return YES;
    }


压缩之后，上传的文件大小只有平时的十分之一，效率提升十分可观。解压的时候需要使用 [`Unarchiver`](https://itunes.apple.com/cn/app/the-unarchiver/id425424353?mt=12) 进行解压，系统自带的解压会有问题。

时间仓促，先这样啦，过年会去咯~