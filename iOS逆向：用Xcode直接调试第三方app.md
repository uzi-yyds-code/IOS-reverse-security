# 更新最新热门文章

*   [逆向入门到进阶项目合集](https://github.com/uzi-yyds-code/IOS-reverse-security)



* **欢迎可以加入iOS高级技术交流群：[1001906160](https://jq.qq.com/?_wv=1027&k=KjioxJty)；群文件可以免费真机包下载，更多中高级进阶资料！**

一般动态调试app时，都是在终端里用lldb直接调试，但是用Xcode的`Attach to Process`也可以连接到真机上的进程进行调试。但是只能调试用自己的证书签名的app。

在Xcode上调试的优点：

*   有UI界面，查看堆栈更直接，可以用Xcode打断点。
*   可以使用`debug gauges`里的Disk和Network等工具。
*   输入lldb命令时有自动补全。
*   可以使用Xcode的`Debug UI Hierarchy`功能，直接查看app的界面布局。
*   可以使用Xcode的`Debug Memory Graph`功能，查看当前内存中存在的所有对象，以及对象之间的引用关系。

步骤如下。

## 1.重签名需要逆向的app

重签名后才能用Xcode attach。而重签名前需要对app进行砸壳。这些步骤就不再重复了。

额外说一句，在砸壳后建议进行一下恢复符号表的操作。恢复符号表后，在调试时就能直接在堆栈中看到方法名，免去了计算偏移量然后在hopper里查找的麻烦。参考：[iOS符号表恢复&逆向支付宝](http://blog.imjun.net/posts/restore-symbol-of-iOS-app/), [restore-symbol](https://github.com/tobefuturer/restore-symbol)。

## 2.Attach to Process

重签名后安装到越狱设备上，启动app，在Xcode中随便打开一个工程，选择越狱设备，就可以在`Debug->Attach to Process`中找到正在运行的进程名和进程id，点击后就会开始连接。大概过1分钟就会连接上。

## 3.查看UI

连接上后，就可以点击使用Xcode的`Debug UI Hierarchy`来查看界面布局：

![image.png](https://upload-images.jianshu.io/upload_images/19704571-66391ff8216b5a62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




注意，`Debug UI Hierarchy`对Xcode版本似乎有要求，在调试重签名app时，Xcode8.3.2可以，而Xcode8.2就没有这个功能按钮。

## 4.查看内存信息

点击`Debug Memory Graph`按钮，可以查看当前内存中存在的数据。打印地址，查看引用关系，可以配合`malloc stack`进行追踪。如果打开了`malloc stack`，就可以直接在右边显示这个对象的创建堆栈。参考：[iOS逆向：在任意app上开启malloc%20stack追踪内存来源](https://zuikyo.github.io/2017/05/04/iOS%E9%80%86%E5%90%91%EF%BC%9A%E5%9C%A8%E4%BB%BB%E6%84%8Fapp%E4%B8%8A%E5%BC%80%E5%90%AFmalloc%20stack%E8%BF%BD%E8%B8%AA%E5%86%85%E5%AD%98%E6%9D%A5%E6%BA%90/)。

![image.png](https://upload-images.jianshu.io/upload_images/19704571-ee797cf6956d864a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 5.查看正在使用的文件

`debug gauges`中的Disk工具可以查看app当前打开的文件。

![image.png](https://upload-images.jianshu.io/upload_images/19704571-fcf40cb5908d91c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 5.instrument调试

类似的，也可以用instrument调试重签名后的app，不过并不是所有工具都可以使用，对逆向的帮助不大。

# 更新最新热门文章

*   [逆向入门到进阶项目合集](https://github.com/uzi-yyds-code/IOS-reverse-security)



* **欢迎可以加入iOS高级技术交流群：[1001906160](https://jq.qq.com/?_wv=1027&k=KjioxJty)；群文件可以免费真机包下载，更多中高级进阶资料！**

作者：黑超熊猫zuik
链接：https://juejin.cn/post/6844903582660034568
