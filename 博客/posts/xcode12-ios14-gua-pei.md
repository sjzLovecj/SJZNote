---
title: 'Xcode12 iOS14 适配'
date: 2020-10-26 10:40:29
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
1. UITableView 或 UICollectionView，点击没反应 或者 视图被遮挡
> 原因: UITableView 或 UICollectionView的Cell视图层级改变，视图加载完成之后，contentView在Cell子视图的最上层，自己添加到Cell上的子控件被contentView遮挡

> 更改：不要将控价直接添加到Cell上，要添加到Cell的contentView上面

2. Xcode12打包上线后，iOS12上滑动UITableView 或 UICollectionView崩溃（自己运行找不到）
> 原因：Xcode12打包上线后，iOS12上大面积崩溃，代码中 返回UITableViewCell 和 UICollectionViewCell的时候，判断最后返回了nil，
> 还有一部分是向已经释放的对象（野指针）发送消息，主要原因 程序存在 内存泄漏(这个应该不算适配问题，自己写代码不注意的问题)

> 依据：崩溃日志中：[Exception Type: EXC_BAD_ACCESS (SIGSEGV)](https://blog.csdn.net/dreamersharon/article/details/49561481)


> 修改：通过Analyze静态分析，将所有可能造成泄漏的地方修改掉

3. Xcode12 打包后，信号量崩溃
> 原因：信号量使用不当，可能因为线程优先级，造成了内存问题
> 修改：没有特别好的办法，去掉了信号量的使用

4. Xcode12 编译第三方库失败
> 报错：building for iOS Simulator, but linking in object file built for iOS
> 原因：原来xcode12模拟器已经用arm架构来编译项目了，而link链接的还是x86架构
> 修改：Target--Build Settings--VALID_ARCHS 中添加需要的架构
            或将VALID_ARCHS删除掉（目前采取 去掉的方式）

5.  私有库中使用.a 或者 .framework，cocoaPods lint失败
  > 报错：Ld /Users/sjz/Library/Developer/Xcode/DerivedData/App-emljzuayowfblgduqgmiktqhgmzy/Build/Intermediates.noindex/App.build/Release-iphonesimulator/App.build/Objects-normal/i386/Binary/App normal i386
> 相似问题：i386 换成 arm64 或者 armv7
   
> 修改：在podspace文件中添加：s.pod_target_xcconfig = { 'VALID_ARCHS' => 'x86_64 armv7 arm64'}
> 还需要将：
> Excluded Architecture 加上 arm64  （这个我承认，我没添加）
> Build Active Architecture Only 设置为 NO

6. 私有库中使用.a 或者 .framework，其他工程引入该私有库，正常运行，打包报错：
```
ld: bitcode bundle could not be generated because '/Users/sjz/Desktop/workPlace/你的工程/Pods/私有库/Pods/BaiduMapKit/BaiduMapKit/BaiduMapAPI_Base.framework/BaiduMapAPI_Base(FontRenderer.o)' was built without full bitcode. All object files and libraries for bitcode must be generated from Xcode Archive or Install build file '/Users/sjz/Desktop/workPlace/你的工程/Pods/私有库/Pods/BaiduMapKit/BaiduMapKit/BaiduMapAPI_Base.framework/BaiduMapAPI_Base' for architecture armv7
```
> 分析
> 1. bitcode bundle could not be generated was built without full bitcode.说明BaiduMapAPI_Base不支持开启bitcode
> 2. 工程 和 私有库中 bitcode 已经全设置成 NO
> 3. 打包还不成功
> 4. 说明 私有库 中bitcode的开启受 有某些 依赖库 的影响

解决方案：在工程的 podfile中添加（在私有库中添加，并没有起到作用）
```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['ENABLE_BITCODE'] = 'NO'
    end
  end
end
```



   