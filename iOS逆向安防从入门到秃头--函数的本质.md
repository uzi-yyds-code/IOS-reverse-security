*   兄弟们，小谷又是勤奋的一天~~

*   本人的高级交流群：1001906160，欢迎各位！

*   今天和兄弟们一起，近距离观察下，`函数之美`~

*   我也就直接步入正题了~

## 1\. 函数的参数

*   兄弟们普遍都听说过：`函数`的`参数`最好不要超过`8`个，`OC方法`的调用不要超过`6`个

*   因为`OC方法`有`2`个`隐藏参数（id self, SEL _cmd）`，所以其实也是`8`个，那么`8`个是怎么来的呢?

*   接下来，兄弟们一探究竟~

> 我们先假设，传入参数为`9`个，看看`汇编`是怎么操作的（这里汇编`不能`用`release`版，否则`LLVM`都会优化掉）

*   上代码：

```
void parameterText(int a,int b,int c,int d,int e,int f,int g,int h,int i){

    printf("%d",a+b+c+d+e+f+g+h+i);

}
int main(int argc, char * argv[]) {
    parameterText(1,2,3,4,5,6,7,8,9);
}
复制代码
```

> 我们断点看`汇编`

> 小谷非常贴心的写好了注释~

*   mian 方法

![1.png](https://upload-images.jianshu.io/upload_images/19704571-e7fc549628db29d2.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   parameterText函数

![2.png](https://upload-images.jianshu.io/upload_images/19704571-10f8260443d668b1.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**我们通过`汇编`可以看出，`x0-x7`存参数，超过之后就会入栈，等待用的时候，会在调用的函数中`偏移取值、在存值、在取值、在相加`**

> 所以，参数不超过8个是有原因的~~ (`x0-x7`存参数)

## 2\. 函数的返回值

*   我们仔细分析了函数的`参数`，下面我们研究下`函数的返回值`。会不会有什么惊喜~

### 2.1\. 普通返回值

*   普通情况下我们只会返回一个值。我们研究下，他是怎么存储的、怎么获取的~

*   来个简单的加法吧

```
int sumTest(int a, int b){
    return a+b;
}
int main(int argc, char * argv[]) {
    int c = sumTest(10,20);
    printf("%d\n",c);
}
复制代码
```

> 老规矩，兄弟们，断点看汇编~

*   main方法汇编~(小谷喜欢紫色和蓝色~)

![3.png](https://upload-images.jianshu.io/upload_images/19704571-e4b72648054b9610.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   sumTest方法汇编

![4.png](https://upload-images.jianshu.io/upload_images/19704571-cd3685474247446d.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 上面分析汇编我们可以看出来— 返回值存放在`w0`中（因为这个值需要4位，要是8位应该就是`X0`中，这个大家可以自己试验下~ ）(毕竟`w0`是`x0`的后低位)

### 2.2\. 结构体返回值

> 来点刺激的~

*   我们刚刚分析了普通的返回值。那我就比较好奇要是`结构体`，这种比较特殊的呢？

*   而且，我突然想到了一个事情。结构体的参数要是`大于8位`会怎么样？，诶~，搞一波搞一波。我这无处安放的求知欲

*   代码走起~

```
struct XGSTRUCT {
    int a;
    int b;
    int c;
    int d;
    int e;
    int f;
    int g;
    int h;
    int i;
};

struct XGSTRUCT getStruct(int a,int b,int c,int d,int e,int f,int g,int h,int i){
    struct XGSTRUCT xgStr;
    xgStr.a = a;
    xgStr.b = b;
    xgStr.c = c;
    xgStr.d = d;
    xgStr.e = e;
    xgStr.f = f;
    xgStr.g = g;
    xgStr.h = h;
    xgStr.i = i;
    return xgStr;
}

int main(int argc, char * argv[]) {
    struct XGSTRUCT xg_str = getStruct(1,2,3,4,5,6,7,8,9);
    printf("%d\n%d\n%d\n%d\n%d\n%d\n%d\n%d\n%d\n",xg_str.a,xg_str.b,xg_str.c,xg_str.d,xg_str.e,xg_str.f,xg_str.g,xg_str.h,xg_str.i);
}
复制代码
```

*   看`汇编代码`喽~

> `mian函数`实在是太长了~（不能截图分析了~）（我把关键的标好注释了~）

```
StackFuncDemo`main:
    0x102a8216c <+0>:   sub    sp, sp, #0xa0             ; =0xa0 
    0x102a82170 <+4>:   stp    x20, x19, [sp, #0x80]
    0x102a82174 <+8>:   stp    x29, x30, [sp, #0x90]
    0x102a82178 <+12>:  add    x29, sp, #0x90            ; =0x90 
    0x102a8217c <+16>:  stur   w0, [x29, #-0x14]
    0x102a82180 <+20>:  stur   x1, [x29, #-0x20]
    0x102a82184 <+24>:  sub    x8, x29, #0x44            ; =0x44 把x8指向 x29(栈底)偏移 0x44（这就相当于预留了一段空间）
    0x102a82188 <+28>:  mov    w0, #0x1
    0x102a8218c <+32>:  mov    w1, #0x2
    0x102a82190 <+36>:  mov    w2, #0x3
    0x102a82194 <+40>:  mov    w3, #0x4
    0x102a82198 <+44>:  mov    w4, #0x5
    0x102a8219c <+48>:  mov    w5, #0x6
    0x102a821a0 <+52>:  mov    w6, #0x7
    0x102a821a4 <+56>:  mov    w7, #0x8
->  0x102a821a8 <+60>:  mov    x9, sp                   ;把x9指向sp(当前函数的栈顶)
    0x102a821ac <+64>:  mov    w10, #0x9                ;把#0x9存入w10
    0x102a821b0 <+68>:  str    w10, [x9]                ;把w10存在x9(当前函数的栈顶)的高地址
    0x102a821b4 <+72>:  bl     0x102a820f0              ; getStruct at main.m:46
    0x102a821b8 <+76>:  ldur   w10, [x29, #-0x44]       ;下面的取值操作就是当时预留的地址
    0x102a821bc <+80>:  mov    x8, x10
    0x102a821c0 <+84>:  ldur   w10, [x29, #-0x40]
    0x102a821c4 <+88>:  mov    x9, x10
    0x102a821c8 <+92>:  ldur   w10, [x29, #-0x3c]
    0x102a821cc <+96>:  mov    x11, x10
    0x102a821d0 <+100>: ldur   w10, [x29, #-0x38]
    0x102a821d4 <+104>: mov    x12, x10
    0x102a821d8 <+108>: ldur   w10, [x29, #-0x34]
    0x102a821dc <+112>: mov    x13, x10
    0x102a821e0 <+116>: ldur   w10, [x29, #-0x30]
    0x102a821e4 <+120>: mov    x14, x10
    0x102a821e8 <+124>: ldur   w10, [x29, #-0x2c]
    0x102a821ec <+128>: mov    x15, x10
    0x102a821f0 <+132>: ldur   w10, [x29, #-0x28]
    0x102a821f4 <+136>: mov    x16, x10
    0x102a821f8 <+140>: ldur   w10, [x29, #-0x24]
    0x102a821fc <+144>: mov    x17, x10
    0x102a82200 <+148>: adrp   x0, 1
    0x102a82204 <+152>: add    x0, x0, #0xf48            ; =0xf48 
    0x102a82208 <+156>: mov    x19, sp
    0x102a8220c <+160>: str    x8, [x19]
    0x102a82210 <+164>: str    x9, [x19, #0x8]
    0x102a82214 <+168>: str    x11, [x19, #0x10]
    0x102a82218 <+172>: str    x12, [x19, #0x18]
    0x102a8221c <+176>: str    x13, [x19, #0x20]
    0x102a82220 <+180>: str    x14, [x19, #0x28]
    0x102a82224 <+184>: str    x15, [x19, #0x30]
    0x102a82228 <+188>: str    x16, [x19, #0x38]
    0x102a8222c <+192>: str    x17, [x19, #0x40]
    0x102a82230 <+196>: bl     0x102a82588              ; symbol stub for: printf
    0x102a82234 <+200>: mov    w10, #0x0
    0x102a82238 <+204>: mov    x0, x10
    0x102a8223c <+208>: ldp    x29, x30, [sp, #0x90]
    0x102a82240 <+212>: ldp    x20, x19, [sp, #0x80]
    0x102a82244 <+216>: add    sp, sp, #0xa0             ; =0xa0 
    0x102a82248 <+220>: ret    
复制代码
```

*   我们主要分析下 `getStruct`函数~

![5.png](https://upload-images.jianshu.io/upload_images/19704571-f79782c83a14c795.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> **看来`结构体的返回值`，是提前`预留`了一块空间，来存的，参数多了，预留空间不会有影响，但是`传值`的时候，还是需要`入栈`操作**

> 果然啊。`LLVM`好生强大~（我以后要研究一下~）

## 3\. 函数的局部变量

*   `局部变量`应该就好看多了（主要是`汇编代码`不会那么多~）

*   上代码~

```
int sumTest(int a, int b){
    int c = 5;
    return a+b+c;
}
int main(int argc, char * argv[]) {
    int d = sumTest(10,20);
    printf("%d\n",d);
}
复制代码
```

> 嘿嘿~，研究问题的时候就是这样，自己不会写，没有关系，看看系统级别的是怎么实现的，让他们给咱们一个思路~

*   我就直接看`sumTest`函数了

![6.png](https://upload-images.jianshu.io/upload_images/19704571-fb4480a852ef7a40.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 看来临时变量也不能被`x0-x7`宠幸啊~

## 4\. 函数的嵌套使用

*   上面函数已经理解的差不多了~。但是，`函数调用栈`——我们最后来感受一波，`嵌套`调用吧~（当然不能死循环啊。😆）

*   直接来个简单的代码 (这个就巨简单的那种~（另外还有`函数的参数嵌套传递`，大家可以自己试试）)

```
void xgNestedTest2(){
    printf("666");
}

void xgNestedTest(){
    xgNestedTest2();
}

int main(int argc, char * argv[]) {
    xgNestedTest();
}
复制代码
```

![7.png](https://upload-images.jianshu.io/upload_images/19704571-9d0b27ee92d77831.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作者：小谷先森
链接：https://juejin.cn/post/6943494136246829069/
