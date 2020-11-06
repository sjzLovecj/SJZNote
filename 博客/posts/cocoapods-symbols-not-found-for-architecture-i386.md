---
title: 'cocoaPods symbol(s) not found for architecture i386'
date: 2020-09-27 14:30:10
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
在 podspec文件中添加：
```
valid_archs = ['armv7s','arm64',]
s.xcconfig = {
  'VALID_ARCHS' =>  valid_archs.join(' '),
}
s.pod_target_xcconfig = {
    'ARCHS[sdk=iphonesimulator*]' => '$(ARCHS_STANDARD_64_BIT)'
}
```
指定支持的框架