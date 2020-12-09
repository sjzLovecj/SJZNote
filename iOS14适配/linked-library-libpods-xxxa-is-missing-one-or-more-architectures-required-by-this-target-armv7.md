能正常真机测试，但build的时候会失败（红色警告）
升级xcode12后，项目在run debug时候是正常运行的，但是在build或者run release的时候就会出现如标题的红色错误。

在网上找到解决方法：

在Target-Build Settings-Excluded Architectures中添加以下代码EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_simulator__NATIVE_ARCH_64_BIT_x86_64=arm64 arm64e armv7 armv7s armv6 armv8 EXCLUDED_ARCHS=$(inherited) $(EXCLUDED_ARCHS__EFFECTIVE_PLATFORM_SUFFIX_$(EFFECTIVE_PLATFORM_SUFFIX)__NATIVE_ARCH_64_BIT_$(NATIVE_ARCH_64_BIT))

参考答案：https://cloud.tencent.com/developer/article/1704754

相关回答： https://stackoverflow.com/questions/63607158/xcode-12-building-for-ios-simulator-but-linking-in-object-file-built-for-ios