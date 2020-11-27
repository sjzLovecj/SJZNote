---
title: '第三章 应用程序网络与能耗优化'
date: 2020-11-25 09:27:35
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
## 3.1 深入iOS网络开发技术
应用程序中网络访问都需要通过URL来进行，iOS中的URL加载系统包含许多类与协议，这些类和协议互相协作完成URL加载的信息配置、协议支持、身份验证、Cookie和缓存功能。![URL加载相关类的关系表](https://sjzlovecj.github.io//post-images/1606269189646.png)

1. NSURLRequest类负责一个具体的网络请求，其内部封装一个请求路径NSURL对象，如果需要对请求参数进行配置，则可以使用NSMutableURLRequest
2. NSURLResponse类封装了请求的响应数据，响应数据包括两部分，一部分是返回数据的状态码，数据长度、编码等信息。另一部分是内容数据本身
3. NSURLCredential、NSURLProtectionSpace、NSURLCredentialStorage、NSURLAuthenticatioChallenge这4个类对请求凭证进行相关设置
4. NSURLCache类用于管理NSURLRequest请求缓存
5. NSHTTPCookieStorage、NSHTTPCookie是用于持久化存储HTTP请求的Cookie数据

### 3.1.1 初识NSURLSession
Session表示会话，一个网络连接的建立可以理解为会话的建立。NSURLSession提供了如下三种类型的网络会话：
- Default类型：提供前台请求相关方法，支持配置缓存、身份凭证等
- Ephemeral类型：即时请求类型，不使用缓存、身份凭证等
- Background：后台类型，支持在后台完成请求任务

在每个网络会话中可以添加多个网络请求任务，网络请求任务也有如下三种类型：
- 数据任务：使用NSData对象进行数据的发送和获取，一般用于短数据的任务。
- 下载任务：下载文件数据，支持后台下载
- 上传任务：以文件的形式上传数据，支持后台上传

通过NSURLSessionConfiguration类对象可以对NSURLSession进行配置与创建
在使用NSURLSession进行网络请求时需要创建NSURLSessionTask对象，每一个NSURLSessionTask对象就是一个请求任务，有两种方式来获取请求返回的数据，一种通过Block，一种通过代理回调

如果进入后台，代理方法将不再被回调，下载完成将调用AppDelegate中的如下方法`application:handleEventsForBackgroundURLSession:completionHandler:`
