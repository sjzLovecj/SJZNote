### TestFlight

- 需要运行在iOS8及以上版本设备上
- 需要安装TestFlight App
- 有效时间（90天）
- 测试人员有最大上限（最多10000）
- TestFlight版本，可以反馈，需要审核（1天左右）
- TestFlight版本，可以生成公开链接，并且可以修改公开链接用于安装（[https://testflight.apple.com/join/xxxxxx](https://links.jianshu.com/go?to=https%3A%2F%2Ftestflight.apple.com%2Fjoin%2Fxxxxxx)），安装链接（[itms-beta://testflight.apple.com/join/xxxxxx](https://links.jianshu.com/go?to=itms-beta%3A%2F%2Ftestflight.apple.com%2Fjoin%2Fxxxxxx)）

整体灰度发布的思路：

![img](image/1200.png)

该流程特点：

1. 可以根据用户id判断用户是否满足我们要发布版本的对象用户
2. 需要判断用户手机是否安装testFlight，否则无法安装testFlight测试版本

该灰度发布实现完全没有使用苹果官方提供的分阶段发布，而是结合testFlight测试版本来实现的。10000安装使用数量，一定程度上已经满足版本的各种测试。

该策略实现的灰度发布，也有些弊端。比如，**安装testflight测试版本出现问题，也无法版本回滚，只有在发布新的更正版本后才能避免bug。**但是在声名“内测”版本，并且安装用户数量在10000以内情况，这种更具有针对性的灰度发布还是很有优势的。

![image-20201215105909888](image/image-20201215105909888.png)

```objective-c
    CGFloat systemVersion = [[[UIDevice currentDevice] systemVersion] floatValue];
    NSURL* invitationpURL = [NSURL URLWithString:@"邀请链接"];
    if (systemVersion>=10.0)
    {
        /*@{UIApplicationOpenURLOptionUniversalLinksOnly : @NO}
         YES: App不存在时，不使用Safari对应的链接
         NO: App不存在时，使用Safari对应的链接
         */
        [[UIApplication sharedApplication] openURL:invitationpURL options:@{UIApplicationOpenURLOptionUniversalLinksOnly : @YES} completionHandler:^(BOOL success) {

        }];
    }else {
        //忽略警告  -Wdeprecated-declarations：警告的类型
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
        //当TestFlight 不存在时，使用Safari对应的链接
        [[UIApplication sharedApplication] openURL:invitationpURL];
#pragma clang diagnostic pop
    }
```



> 注意
>
> - 对于外部测试者，测试包需要经过Apple的审核。对于内部测试者则无此限制。
> - 第一次提交版本需要进行全面审核，审核通过后才可以邀请外部测试者进行测试。之后如果有很大跟新的版本才需要再次全面审核，小更新的版本可以不用审核。
> - 应用的Bate版本上传后有90天的有效期，逾期后如果没有发布新的版本，测试者将无法运行测试版应用程序。
> - 发布测试版可以包含相关指导信息，例如新功能，测试点等。每当发布新版本，相关的测试者都能通过TestFlight获得通知。
> - 最多同时测试100个App
> - 测试版如果经测试没有问题，可直接提交审核上架App Store。
>
> 1. 内部测试者
>
>    - 内部测试者必须在iTunes Connect中你的Team里注册成为Admin，App Manager，Developer或者Marketer角色的成员。
>    - 每个app最多可添加25个内部测试者
>    - 每个内部测试者可以最多有30台测试设备
>    - 内部测试者可以访问可供测试的全部Beta Builds
>
> 2. 外部测试者
>
>    - 可以通过邮件邀请外部测试
>    - 最多可邀请10000名外部测试者
>    - 外部测试者可以访问开发者对其发布的Beta Builds。
>    - 在iTunes Connect中可以将外部测试者分组，并对不同的组分发不同版本的Beta Builds。
>    - 如果已经安装TestFlight应用，那么会在应用内部看到可以测试的app，如果没有安装，会打开一个网页，提示用户安装TestFlight应用，并提供了一个待测试app的邀请码
>
> 3. 关于IAP
>
>    Apple文档中，所有的IAP在测试期间是免费的，所有在测试期间进行的IAP购买均不会延续到正式版中。