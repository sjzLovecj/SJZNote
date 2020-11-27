---
title: '将网站数据获取到内存中'
date: 2020-11-26 13:45:09
tags: [URL加载系统]
published: true
hideInList: false
feature: 
isTop: false
---
### 概述
通过从URL会话创建数据任务，直接接收数据到内存中。

对于与远程服务器的小型交互，可以使用URLSessionDataTask类将响应数据接收到内存中（与使用URLSessionDownloadTask类不同，后者将数据直接存储到文件系统中）。数据任务非常适合调用Web服务端点之类的用途。

您使用URL会话实例创建任务。如果您的需求很简单，则可以使用URLSession类的共享实例。如果要通过委托回调与传输进行交互，则需要创建一个会话，而不是使用共享实例。您在创建会话时使用URLSessionConfiguration实例，同时传入实现URLSessionDelegate或其自协议之一。会话可以重复使用来创建多个任务，因此对于所需的每个唯一配置，创建一个会话并将其存储为属性。

> 注意
> 不要创建超出您需要的会话。例如，如果您的应用程序有几部分需要类似配置的会话，请创建一个会话并在他们之间共享。

建立会话后，可以使用dataTask初始化方法之一创建数据任务。任务以挂起状态创建，可以通过调用resume方法启动

### 使用完成处理程序接收结果
获取数据的最简单方法是创建一个使用完成处理程序的数据任务。通过这种安排，任务将服务器的相应、数据以及可能的错误传递到您提供的完成处理程序块。下图显示了会话和任务之间的关系，以及如何将结果传递给完成处理程序。
图 创建完成处理程序以接收任务结果
![](https://sjzlovecj.github.io//post-images/1606360444093.png)
要创建使用完成处理程序的数据任务，请调用URLSession的dataTask(with:)方法。您的完成处理程序需要做三件事：
1. 验证错误参数为nil，如果不是，则说明发生了传输错误；处理错误并退出
2. 检查响应参数以验证状态码指示成功，并且MIME类型是期望值。如果不是，请处理服务器错误并退出
3. 根据需要使用数据实例

下面代码 显示了用户获取URL内容的startLoad()方法。首先使用URLSession类的共享实例创建一个数据任务，将其结果传递给完成处理程序。检查本地和服务器错误后，此处理程序将数据转换为字符串，并使用其填充WKWebView。当然，您的应用程序可能还有其他用途来获取数据，例如将其解析为数据模型。
```
func startLoad() {
    let url = URL(string: "https://www.example.com/")!
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            self.handleClientError(error)
            return
        }
        guard let httpResponse = response as? HTTPURLResponse,
            (200...299).contains(httpResponse.statusCode) else {
            self.handleServerError(response)
            return
        }
        if let mimeType = httpResponse.mimeType, mimeType == "text/html",
            let data = data,
            let string = String(data: data, encoding: .utf8) {
            DispatchQueue.main.async {
                self.webView.loadHTMLString(string, baseURL: url)
            }
        }
    }
    task.resume()
}
```
> 重要
> 完成处理程序的创建和调用是在不同的Grand Central Dispatch队列上的。因此，任何使用数据或错误来更新UI工作都应明确放置在住队列中。

### 与代理一起接收传输详细信息和结果
为了再任务进行时对任务活动更大程度的访问，在创建数据任务时，您可以在会话上设置委托，而不是提供完成处理程序。下图显示了这种模式
图 实现委托以接收任务结果
![](https://sjzlovecj.github.io//post-images/1606368072926.png)

通过这种方法，部分数据会在到达时提供给URLSessionDataDelegate的urlSession(_:dataTask: didReceive:)方法，直到传输完成或失败并出现错误。随着转移的进行，委托还接收其他类型的事件。

使用委托时，您需要创建自己的URLSession实例，而不是使用URLSession类的简单共享实例。创建一个新的会话是您可以将自己的类设置为会话的委托，如下列代码所示。

声明您的类实现一个或多个委托协议（URLSessionDelegate，URLSessionTaskDelegate，URLSessionDataDelegate和URLSessionDownloadDelegate）。然后使用初始化程序init(configuration: delegate:delegateQueue:)创建URL会话实例。您可以定制与此初始化程序一起使用的配置实例。例如，将waitsForConnectivity设置为true是个好主意，这样，会话将等待适当的连接，而不是在所需的连接不可用时立即失败。

程序 创建使用委托的URLSesstion
```
private lazy var session: URLSession = {
    let configuration = URLSessionConfiguration.default
    configuration.waitsForConnectivity = true
    return URLSession(configuration: configuration,
                      delegate: self, delegateQueue: nil)
}()
```

下面代码 显示了一个startLoad()方法，改方法使用此会话启动数据任务，并使用委托回调处理接收到的数据和错误。此代码实现了三个委托回调：
- `urlSession(_:dataTask:didReceive:completionHandler:)`验证相应是否具有成功的HTTP状态码，并且MIME类型是text/html或text/plain。如果情况并非如此，则任务将被取消；否则，允许继续执行
- `urlSession(_:dataTask:didReceive:) `接收任务接收到的每个Data实例，并将其添加到一个称为ReceivedData的缓冲区中
- `urlSession(_:task:didCompleteWithError:) `首先查看是否发生了传输级错误。如果没有错误，它将尝试将接收到的数据缓冲区转换为字符串并将其设置为webView的内容。

程序 使用带有URL会话数据任务的委托
```
var receivedData: Data?

func startLoad() {
    loadButton.isEnabled = false
    let url = URL(string: "https://www.example.com/")!
    receivedData = Data()
    let task = session.dataTask(with: url)
    task.resume()
}

// delegate methods

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse,
                completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
    guard let response = response as? HTTPURLResponse,
        (200...299).contains(response.statusCode),
        let mimeType = response.mimeType,
        mimeType == "text/html" else {
        completionHandler(.cancel)
        return
    }
    completionHandler(.allow)
}

func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    self.receivedData?.append(data)
}

func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    DispatchQueue.main.async {
        self.loadButton.isEnabled = true
        if let error = error {
            handleClientError(error)
        } else if let receivedData = self.receivedData,
            let string = String(data: receivedData, encoding: .utf8) {
            self.webView.loadHTMLString(string, baseURL: task.currentRequest?.url)
        }
    }
}
```
各种委托协议提供了以上代码中所示方法之外的方法，用于处理身份验证，重定向 和 其他特殊情况。在URLSession讨论中，使用URL会话描述了在传输过程中可能发生的各种回调。