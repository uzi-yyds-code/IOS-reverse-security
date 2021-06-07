*   本人的高级交流群：1001906160，欢迎各位！

*   兄弟们--今天有时间研究这个`switch`了。哪里有不对的地方，大家可以指指点点

## 1\. `switch-case`简单分析

*   1.  首先。我们写一个简单的`switch`语句

```
void judgeSwitch(int a){
    switch (a) {
        case 10:
            printf("我是小明\n");
            break;
        case 2:
            printf("我是小白\n");
            break;
        case 7:
            printf("我是小黑\n");
            break;

        default:
            printf("我是小谷\n");
            break;
    }
}
复制代码
```

*   2.  在好多人眼里，`switch-case`和`if-else`差不多，都是个判断选择~
*   3.  但是也流传着一句话：`switch-case`比`if-else`好

> 哦吼~，我们今天找到了主要目标了，为什么会流传着这么一句话？

## 2\. `switch-case`汇编分析

> 还是上面的代码~

*   1.  调用： `judgeSwitch(5);`
*   2.  我们通过汇编看下`switch-case`和`if-else`有什么异同？

![1.png](https://upload-images.jianshu.io/upload_images/19704571-7ecd65232d9ce7e8.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我们`分析汇编`看到：还是`不停的判断`，`不停的跳转`。

> 感觉和`if-else`没啥区别~

*   3.  疑问：但是上面的为啥要让`传入的参数`减去`0x2`呢？（我怀疑是`最小的case`，但是我没有证据~）

### 2.1\. `case`个数为4个

> 既然没有证据，那就寻找下证据~

*   1.  这个时候，我加入了一个比`2`还小的`case`(也就是`1`了~)
*   2.  代码

```
void judgeSwitch(int a){
    switch (a) {
        case 10:
            printf("我是小明\n");
            break;
        case 2:
            printf("我是小白\n");
            break;
        case 7:
            printf("我是小黑\n");
            break;
        case 1:
            printf("我是小芳\n");
            break;

        default:
            printf("我是小谷\n");
            break;
    }
}
复制代码
```

> 把小芳加入，看看效果是不是减一了~

#### 2.1.1 `switch-case` 的`D炸天`操作

*   1.  我们继续上面的问题：看`汇编代码`是不是`减一`了

![2.png](https://upload-images.jianshu.io/upload_images/19704571-a9456c54fb256b6d.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 他果然是`减一`了

*   2.  但是这个时候我们发现了新的情况:与`if-else`不同了,粗略看好像是通过`一次比较`--然后`地址偏移取值`~他怎么做到了？
*   3.  我们带着疑问，仔细分析一波~

![3.png](https://upload-images.jianshu.io/upload_images/19704571-a304c770b610e8d3.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   4.  不经意间再次感叹`LLVM`的强大

## 3\. `switch-case`流程总结

*   小谷非常细心的画了一波流程图（有点丑~兄弟们不要取笑我）

![4.png](https://upload-images.jianshu.io/upload_images/19704571-20f4db7161e8340f.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4\. 总结

*   1.  兄弟们常说的`switch-case`比`if-else`好是有原因的~
*   2.  `switch`的条件判断的`区间`是有限制有规律的，不能差距过大
*   3.  `case个数`只有`大于等于4`个的时候才会比`if-else`有优势
*   4.  `switch`的参数是我们定义的，`case`也是我们开发者写的----所以，兄弟们可以搞起来了
*   5.  当然，最后一步要感谢了! 哈哈哈，就希望和兄弟们共同进步吧。

作者：小谷先森
链接：https://juejin.cn/post/6946119030323281950/

