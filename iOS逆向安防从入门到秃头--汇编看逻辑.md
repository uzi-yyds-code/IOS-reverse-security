*   本人的高级交流群：1001906160，欢迎各位！
*   小谷又来了~ 今天我们主要看下`if`、`do-while`、`while`、`for`等逻辑(`switch`后面的博客分析，这个有点特殊~)

*   这一次就不一直看源码了，引进个逆向是常用的工具————`Hopper`

## 1\. `Hopper`的使用

> 这个兄弟们用免费的~，毕竟能省点是省点~哈哈哈

*   1.  这个工具就很智能和简单、首先看下他的图标。（下载的时候不要下载错了）

![1.png](https://upload-images.jianshu.io/upload_images/19704571-538614844bc79f5a.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   2.  打开、然后把`二进制文件`拖进去。就OK了~
*   3.  我直接在网上下载了`ipa`(也不知道是谁的~),如下图

![2.png](https://upload-images.jianshu.io/upload_images/19704571-7420fa74c42908d2.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2\. 判断

### 2.1\. 全局变量

*   1.  准备工作已经做完了~，咱们先了解下`全局变量`（这个用到的地方还是挺多的~）
*   2.  老规矩了~

```
int g = 10;

int funcIfLogic(int a, int b){
    return a+b+g;
}

int main(int argc, char * argv[]) {
    int c = funcIfLogic(1,2);
    printf("%d",c);
}
复制代码
```

*   3.  看`汇编、分析逻辑`

![3-1.png](https://upload-images.jianshu.io/upload_images/19704571-cd2a9f0e9a1a9088.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 从分析中可以看到，`全局函数`的取值通过`地址偏移`

> **0x104b1e1f8—(`右移三位，其实就是后3位抹零`)—>0x104b1e—(`加上偏移值(7),在左移三位`)—>0x104b25000—(`加上偏移地址0x554`)—>0x104b25554**

### 2.2\. `if`判断

*   稍微改下代码（加个`if`）

```
int g = 10;

int funcIfLogic(int a, int b){
    if (a > b) {
        return a+g;
    }else{
        return b+g;
    }
}

int main(int argc, char * argv[]) {

    int c = funcIfLogic(1,2);
    printf("%d",c);
}
复制代码
```

*   感觉汇编多了点东西：

![4.png](https://upload-images.jianshu.io/upload_images/19704571-1b01894a576a9fea.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.2.1\. 判断扩展（b指令）

*   上面，看到了`b.le`是`小于等于`的意思。我把常用的命令和意思给兄弟们列一下子~

![5-1.png](https://upload-images.jianshu.io/upload_images/19704571-a9da0ec2ffdc2cff.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.3\. `汇编`转高级`伪代码`~

> 既然我们上面已经介绍了工具，那我们用工具写下逻辑伪代码~

*   1.  首先把二进制文件拖到`hopper`，然后找到方法~

![6.png](https://upload-images.jianshu.io/upload_images/19704571-9d15e7de93d46aee.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   2.  然后兄弟们看我`分析流程`

![7.png](https://upload-images.jianshu.io/upload_images/19704571-27a52e6b2dfd4c25.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我感觉`汇编`难的原因就是因为它枯燥，所以兄弟们要提起裤子来（不对、提起精神来~）

## 3\. 循环

*   接下来，兄弟们一起看下循环~

### 3.1\. `do-while`循环

*   老规矩了~，研究这个东西，我就看看他是如何实现的~

```
void funcLoop(){

    int i = 1;
    do {
        i++;
    } while (i<10);
    printf("i==%d",i);

}
int main(int argc, char * argv[]) {
    funcLoop();
}
复制代码
```

*   `汇编`分析

![8.png](https://upload-images.jianshu.io/upload_images/19704571-b52e00315d3daf37.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**这个是先加，然后判断，然后跳转**

> 这次就和兄弟们一行一行的读了，都差不多。（我把关键的地方拿出来了~）

### 3.2\. `while`循环

*   我们看下`while`的

```
void funcLoop2(){
    int i = 1;

    while (i<10) {
        i++;
    }
    printf("i==%d",i);

}
复制代码
```

> 我们看看他们之间有什么异同~

![9.png](https://upload-images.jianshu.io/upload_images/19704571-e86fc6b7035d7eba.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**`while`循环是先判断，后加值,在跳转**

### 3.3\. `for`循环

*   `for`循环看起来有有点高端的样子。我们看下~

```
void funcLoop3(){
    int i;
    for (i = 1; i < 10; i++) {

    }
    printf("i==%d",i);
}
复制代码
```

*   继续`汇编`

![10.png](https://upload-images.jianshu.io/upload_images/19704571-1a18bd59d8998d6b.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这个逻辑和while循环一样啊，看起来都这么相同。都是，先判断，后加值，在跳转~

## 4\. 总结

> 又到了总结的时候了

*   1.  这次没有弄清楚`switch`语句，我弄明白了必须搞一篇博客，让兄弟们指指点点😆
*   2.  这次研究这个`还原伪代码`，的确会下点功夫，主要还是比较枯燥，我给兄弟们加油
*   3.  最后，希望跟兄弟们共同进步。⛽️（这个是加油站😆）

作者：小谷先森
链接：https://juejin.cn/post/6945066376050507812/

