---
title: '将数据上传到网站'
date: 2020-11-26 13:50:29
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
## 概览
许多应用程序都与服务器一起使用，这些服务器接受图像或文档之类的文件上传，或者使用接收结构化数据的Web服务的API端点，例如JSON。要从您的应用上传数据，您可以使用URLSession实例来创建URLSessionUploadTask实例。上传任务使用URLRequest实例，该实例详细说明如何执行上传。

## 准备上传数据
要上传的数据可以是文件、流或数据的内容，如以下代码所示。
许多Web服务端点都采用JSON格式的数据，您可以通过对数组和字典等可编码类型使用JSONEncoder类创建该对象。如以下代码所示，您可以声明一个符合Codable的结构，创建此类型的实例，然后使用JSONEncoder将实例编码为JSON的数据进行上传。
代码 准备JSON数据上传
```
struct Order: Codable {
    let customerId: String
    let items: [String]
}

// ...

let order = Order(customerId: "12345",
                  items: ["Cheese pizza", "Diet soda"])
guard let uploadData = try? JSONEncoder().encode(order) else {
    return
}
```
创建数据实例还有许多其他方法，例如将图像编码为JPEG或PNG数据，或使用UTF-8等编码将字符串转换为数据。
## 配置上传请求
上传任务需要URLRequest实例。如下代码所示，根据服务器的支持和期望，将请求的httpMethod属性设置为“POST”或“PUT”。使用setValue(_: forHTTPHeaderField:)方法设置要提供的任何HTTP请求头的值，Content-Length等除外。改会话会根据数据大小自动计算出内容长度。

代码 配置URL请求
```
let url = URL(string: "https://example.com/post")!
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Type")
```
## 创建并启动上传任务
要开始上传，请在URLSession实例上调用uploadTask(with:from:completionHandler:)以创建一个上传URLSessionTask实例，并传入请求和您先前设置的数据实例。由于任务以挂起状态开始，因此您可以通过在任务上调用resume方法来开始网络加载过程。以下代码使用共享的URLSession实例，并在完成处理程序中接收其结果。处理程序在使用任何返回的数据之前检查传输和服务器错误。

代码 启动上传任务
```
let task = URLSession.shared.uploadTask(with: request, from: uploadData) { data, response, error in
    if let error = error {
        print ("error: \(error)")
        return
    }
    guard let response = response as? HTTPURLResponse,
        (200...299).contains(response.statusCode) else {
        print ("server error")
        return
    }
    if let mimeType = response.mimeType,
        mimeType == "application/json",
        let data = data,
        let dataString = String(data: data, encoding: .utf8) {
        print ("got data: \(dataString)")
    }
}
task.resume()
```

## 或者， 通过设置委托上传
作为完成处理程序方法的替代方法，您可以在您配置的会话上设置委托，然后使用uploadTask（with：from :)创建上传任务。 在这种情况下，您将实现URLSessionDelegate和URLSessionTaskDelegate协议中的方法。 这些方法接收服务器响应以及任何数据或传输错误。









