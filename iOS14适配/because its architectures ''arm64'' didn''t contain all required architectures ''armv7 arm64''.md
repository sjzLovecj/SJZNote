这是今天遇到的一个BUG,是在Debug下,
because its architectures 'arm64' didn't contain all required architectures 'armv7 arm64'

原因是cocoapods下依赖

只要把Build Settings下的  Build Active Architecture Only  设置为YES就可以了

## 可以解决  依赖第三方库 找不到头文件的问题

Build Active Architecture Only：是否只编译当前设备适用的指令集（如果这个参数设为YES，使用iPhone 6调试，那么最终生成的一个支持ARM64指令集的Binary。一般在DEBUG模式下设为YES，RELEASE设为NO）

将Debug设置为YES,Release设置为NO。若两个都设置为YES上架打包用iPhone5s以上的手机编译发布包时不会支持iPhone5s以下的设备；用iPhone5以下的手机打包时的ipa包不包含64位。