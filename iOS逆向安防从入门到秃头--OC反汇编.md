*   本人的高级交流群：1001906160，欢迎各位！
*   兄弟们，我回来了~

*   前面和兄弟们写了好多`汇编`的知识，今天我们开始步入正题了：`OC的汇编`

## 1\. 方法的调用

*   1.  我们开始就简单写个`OC对象`，看下他的汇编吧

```
@interface XGPerson : NSObject
+(XGPerson *)person;
@end

@implementation XGPerson
+ (XGPerson *)person{
    return [self alloc];
}
@end

int main(int argc, char * argv[]) {
    XGPerson *p = [XGPerson person];
}
复制代码
```

*   2.  大家应该知道方法的本质就是消息的发送:`objc_msgSend`

### 1.1\. 动态分析

*   1.  我们先看下`汇编代码`

![1.png](https://upload-images.jianshu.io/upload_images/19704571-5cde86515eb8d2d0.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我们知道`objc_msgSend`会有2个默认的参数`（id self, SEL _cmd）`

> **这个根据前面的知识我们就更能理解，为什么参数最好控制在6个以内了**

*   2.  我们可以动态看下`x0,x1`的值

![2.png](https://upload-images.jianshu.io/upload_images/19704571-ed697e9f47e0a441.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.2\. 静态分析

> `动态调试`是舒服，但是我们`逆向开发`的时候好多时候都会`静态分析`~

*   1.  我们还是要老规矩分析一波~

![3-1.png](https://upload-images.jianshu.io/upload_images/19704571-ac3c00bab4246dcc.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   2.  然后我们验证我们分析的正确性（`iOS是小端模式`，所以取出地址，`从右向左`读）

![4.png](https://upload-images.jianshu.io/upload_images/19704571-f010f65f66bc4900.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 看来我们的`静态分析`没有错

#### 1.2.1\. 工具分析

> 说实在的：`静态分析`一个方法，我人都快傻掉了。要是真正的工程（成千上万的方法），我不瓦特了？

> 估计那些·啥啥家·也是真想的~

*   咱们先用`Hopper`看下`二进制文件`，会不会效果好点

![5.png](https://upload-images.jianshu.io/upload_images/19704571-0bd0c7ce86b497ca.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 那些`imp`指向的方法的实现，其实都是`objc源码`里面的方法---很早之前写了一篇博客[objc源码调试](https://juejin.cn/post/6868929959076265997) （目前最新的是`objc4-818.2`，其实差不多）

## 2\. `block`反汇编

> 关于`block`，我也写了一篇博客[block底层分析](https://juejin.cn/post/6902030826855202824/) --- 实不相瞒，太早了，有点忘记了，不过应该还可以参考

*   1.  不过曾经没有看过`汇编`。现在看`汇编`又可以明白好多东西
*   2.  先写一些代码

```
int main(int argc, char * argv[]) {
    void(^block)(NSInteger index) = ^(NSInteger index){
        NSLog(@"block -- %ld",index);
    };

    block(1);
}
复制代码
```

*   3.  我稍稍的画了个小小的图(小谷艺术细菌比较少，兄弟们多担待~)

![6.png](https://upload-images.jianshu.io/upload_images/19704571-5b1dc368e7c2b09e.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我的理解：`block`其实也是个对象 --- 就是有点特殊

## 3\. 总结

*   `hopper`是专门做`OC的反汇编`之类的。但是我们项目中好多都会有`C++和C`代码，而且这个`伪代码`不太友好 --- 以后可能会用一个其他的工具

*   写了好多`汇编`的博客，其实就那么些`指令`。我需要的时候就是一边查着看--接下来就要搞搞传说中的`逆向`了~

*   还有`谢谢`兄弟们的`点赞和浏览`，坚持学习到了现在，非常真诚的给兄弟们`鞠个躬Thanks♪(･ω･)ﾉ`

*   好了！兄弟们，等待我的下一篇产出 ~

作者：小谷先森
链接：https://juejin.cn/post/6949054730827989029/
