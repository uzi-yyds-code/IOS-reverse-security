*   本人的高级交流群：1001906160，欢迎各位！


*   日常开发中，兄弟们或多或少都会用到加密。今天稍微聊一下`RSA&HASH`

## 1\. RSA 非对称加密

### 1.1\. RSA 的特点

*   `RSA`加密的效率比较低（`公钥加密-私钥解密`、`私钥加密-公钥解密`）

*   `RSA`加密的安全性还是比较高的(由于中间传输的数据都是加密数据、不好破解)

### 1.2\. iOS中使用 RSA 非对称加密

> 在`iOS`中使用`公钥和私钥`。是`.der`和`.p12`文件。

#### 1.2.1\. `.der`和`.p12`文件文件生成

*   1.  生成秘钥文件`private_key.pem`(这里也可以写成`2048`字节，不过一般都是`1024`字节)

`openssl genrsa -out private_key.pem 1024`

![1.png](https://upload-images.jianshu.io/upload_images/19704571-a77a7e3de7bfdb24.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   2.  请求文件`csr`的生成(通过私钥文件)

`openssl req -new -key private_key.pem -out rsacert.csr`

![2.png](https://upload-images.jianshu.io/upload_images/19704571-8298743f713aeb51.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 填一些请求信息

*   3.  获取自签文件`crt`

`openssl x509 -req -days 3650 -in rsacert.csr -signkey private_key.pem -out rsacert.crt`

![3.png](https://upload-images.jianshu.io/upload_images/19704571-0f28b27c97930120.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这个有效期是10年

*   4.  通过`crt`获取公钥文件`.der`(在iOS开发中使用)

`openssl x509 -outform der -in rsacert.crt -out rsacert.der`

*   5.  `iOS`中使用的私钥`.p12`文件生成（这里设置的密码要记住）

`openssl pkcs12 -export -out p.p12 -inkey private_key.pem -in rsacert.crt`

![4.png](https://upload-images.jianshu.io/upload_images/19704571-544cb4ec3513ffa0.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   6.  文件展示一波~

![5.png](https://upload-images.jianshu.io/upload_images/19704571-b5dd3e1b0e7f881b.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.2.2\. RSA代码测试

![6.png](https://upload-images.jianshu.io/upload_images/19704571-5bfee0bb45899bd1.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 上面其实就是`RSA`的用法。一般情况就是`客户端（client）保存公钥`，`服务器（server）保存私钥`，用来加密关键数据（这个数据`不能是大数据`，因为`RSA效率并不高`）

## 2\. HASH 加密

*   兄弟们平常开发中用到的`MD5`其实就是一种`HASH`

> `HASH`是一种思想，他包括`MD5`，但不是`MD5`

### 2.1\. HASH 的特点

*   `HASH`是`不可逆运算`（意思就是变了之后就变不回来了）

> 传说中的HASH找值[CMD5](https://www.cmd5.com/),是人家搜集了成千上万的数据，去查表。

*   `HASH`经常用在`密码加密`（这个兄弟们应该都用过的）

*   `HASH`加密之后长度一样的，32个字符（16进制表示的）

*   主要用于验证数据

### 2.2\. 代码使用

![7.png](https://upload-images.jianshu.io/upload_images/19704571-ddefe5bf18eccf47.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 具体的实现，网上太多了，这里就不说了

### 2.3\. 加盐

> 由于[CMD5](https://www.cmd5.com/)网站的出,使得单纯的密码没有想象中的那么安全

*   所以我们想到了`加盐`~(所谓的`加盐`就是给他拼接一段数据)

![8.png](https://upload-images.jianshu.io/upload_images/19704571-6568715a4cba3363.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.4\. `HMAC加盐`

> 上面`加盐`写在客户端是固定的，所以说所有用户就公用一个`盐`，还是有点不好

*   现在用的比较多的还是`HMAC`（据说比较安全，我工程中还没有用到~哈哈哈）

![9.png](https://upload-images.jianshu.io/upload_images/19704571-542c077e7d321637.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 其实这也算是`加盐`~ 不过这个`盐`，是从`服务器上取得`，就相对安全了点

## 3\. 密码登录

> 说了一些加密方式（`对称加密`以后在说~）

### 3.1\. 项目中的登录

**`兄得们 都是知道的，用户的密码，开发者绝对不能知道的。`**

*   先说下我们项目的登录（兄弟们，大大们，我说完别攻击啊，要不我就是罪人了~（不过也没啥好攻击的~小破公司））

*   我们公司就是典型的`账号密码`登录

*   密码传给服务器的是`HASH`

*   存入数据库的是`HASH`

*   验证的是`HASH`

> 都是`HASH`

### 3.2\. 自己理想设计的登录

*   男人吗~ 都会有点理想，奈何我职位比较低，说了也不算，就只能和兄弟们在这个口嗨下~哈哈哈

> 我没有办法测试。leader用30秒怼回来了~（原因是：1.现在的还可以用、2、IPA的DAU就那么一点，不用搞这么复杂~：他说服我了，我觉得他说的对~）），诶呀，我这个人就是太容易听取别人的意见了~

*   但是我还是要把自己的想法写出来（兄得们看看好不好~，不好的话希望可以指点下~）

> 虽然可能还是欠缺很多，但是我 感觉只`HASH`还是不太安全，毕竟有那个网站

#### 3.2.1\. 注册流程

![10.png](https://upload-images.jianshu.io/upload_images/19704571-5f188b3a56eae757.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.2.2\. 登录流程

![11.png](https://upload-images.jianshu.io/upload_images/19704571-a35f5ecf749abe4c.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.2.3\. 用户换手机的情况

![12.png](https://upload-images.jianshu.io/upload_images/19704571-74362bf49ce6f5aa.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这个是我想像的，但是我能力和权利都有限，不能试验。以后的幸福生活就靠兄弟们了~

## 4\. 数字签名

*   `数字签名`就好说点了~

*   感觉在做过`支付`的兄弟们面前有点`班门弄斧`了~

*   简单来说就是把重要数据传过来的时候，把`数据的HASH值`带上，在用`RSA加密HASH值`，保证数据的可靠性（防止拿到被人篡改的信息）

*   小谷还是要画图

![13.png](https://upload-images.jianshu.io/upload_images/19704571-b15221febd36fd2a.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 如果中间数据被篡改过，那么客户端对比的`HASH`就不同，这段数据我们就会丢弃不用~

## 5\. 总结

*   开发中用到`RSA&HASH加密`的不少。可以稍微了解下

*   希望这些知识给大家带来一些帮助吧~（小谷是个汉子，不太会说矫情的话，哈哈哈）


作者：小谷先森
链接：https://juejin.cn/post/6950845841045192735/
