来自作者：小谷先森

文章涉及的iOS、资源、源码、真机包、插件提供大家[下载地址](https://jq.qq.com/?_wv=1027&k=RYk46azM)
> 有些`app`也会做`反越狱操作`和`防越狱插件`

> 不过小谷觉得没有必要这么绝。毕竟人家是越狱手机的话，直接掐死就不太好了~

*   小谷今天说下`防盗版`吧

## 1\. 检查 BundleID

*   `BundleID`也称`APPID`，每个应用独一份（其实他的本名是`文件ID`）

*   兄弟们知道用`MonkeyDev重签名`的话，会更改`BundleID`。

*   直接上代码了

```
- (void)checkBundleId{
    NSDictionary *infoDic = [[NSBundle mainBundle] infoDictionary];
    NSString *bundleID = [infoDic objectForKey:@"CFBundleIdentifier"];

    if (![bundleID isEqualToString:@"服务器请求的bundleID"]) {
        //防护措施
        NSLog(@"防护措施");
    }
}
复制代码
```

## 2\. 检查重签名

> `钉钉`好像是用到这个了。

*   检查`重签名`的原理:是检查权力配置文件`embedded.mobileprovision`的 `UUID`和`账户的前缀`

*   如果应用被重签名，`账户的前缀`和 `UUID`会不一样。所以我们可以通过这种方法检查重签名

> 我估计兄弟们都玩过这个

*   由于`embedded.mobileprovision`不是标准的`plist`文件。所以我们要通过处理把他变成`plist`文件，然后在检测

> 上代码了

```
//检测重签名
- (BOOL)isResign{
    NSString * embedProPath = [[NSBundle mainBundle] pathForResource:@"embedded" ofType:@"mobileprovision"];

    NSError *error;
    NSString *embedProStr = [NSString stringWithContentsOfFile:embedProPath encoding:NSISOLatin2StringEncoding error:&error];

    if (embedProStr) {
        NSScanner *scanner = [NSScanner scannerWithString:embedProStr];

        NSString * contentStr;
        BOOL result = [scanner scanUpToString:@"<plist" intoString:&contentStr];

        if (result) {
            result = [scanner scanUpToString:@"</plist>" intoString:&contentStr];

            if (result) {
                //格式化字符串
                NSString *strPlist = [NSString stringWithFormat:@"%@</plist>",contentStr];
//                NSLog(@"plist --> %@",strPlist);

                //plist --> dic
                NSData * data = [strPlist dataUsingEncoding:NSUTF8StringEncoding];
                NSError *errordic;
                NSDictionary *dicPlist = [NSPropertyListSerialization propertyListWithData:data options:0 format:0 error:&errordic];
                //如果没有错误
                if (!error) {
                    //可以查看UDID和签名前缀
                    NSLog(@"dicPlist --> %@",dicPlist);
                    //UUID
                    NSString *UUIDStr = [dicPlist objectForKey:@"UUID"];
                    //签名前缀
                    NSArray *prefixArr = [dicPlist objectForKey:@"ApplicationIdentifierPrefix"];
                    NSString *appPrefix = [prefixArr firstObject];

                    //判断 这里可以做一个服务器请求，具体判断根据自己的项目来！！
                    if ([UUIDStr isEqualToString:@"7a016e01-31c1-4f6e-bfee-998245a9063a"] && [appPrefix isEqualToString:@"9536852PXS"]) {

                        //说明没有重签名
                        NSLog(@"没有重签名");
                        return false;
                    }else{
                        NSLog(@"重签名应用");
                        return true;
                    }

                }
            }
        }
        return true;
    }
    return false;
}
复制代码
```

*   给兄弟们看下`dicPlist`的打印

![1.png](https://upload-images.jianshu.io/upload_images/19704571-f85bf1de8097a075.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这个就可以防止重签名应用了

## 3\. 检查来源（是否来自App Store）

*   还有一种`更帅气的检测`。（偷偷的告诉兄弟们，这个小谷曾经在用，感觉不错~，不过我们公司的`iOS包`不好过审(什么版号的问题)，然后小谷只能把这个去掉了~）

*   `App Store`下载的应用，在 `loadCommand` 中的 `LC_ENCRYPTION_INFO_64`段会有加密情况

*   如果没有加密，一定`不是从App Store下载`的，或者逆向大佬操作过了

*   给兄弟们对比下`加密`和`未加密`的

![2.png](https://upload-images.jianshu.io/upload_images/19704571-5bc25111499cbe78.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![3.png](https://upload-images.jianshu.io/upload_images/19704571-df644488dfda7d28.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我们的`砸壳`，也会把这个值改掉

*   大体的逻辑和判断兄弟们应该都知道了，那么，上代码(兄弟看过`dyld源码`或者`fishhook的源码`就比较好理解查找了~)

```
#include <execinfo.h>
#import <mach-o/ldsyms.h>

//是否是从App Store下载的(这里小谷只判断LC_ENCRYPTION_INFO_64，毕竟现在大多数都不主持32位了)
- (BOOL)isAppStore{
     const uint8_t *command = (const uint8_t *) (&_mh_execute_header + 1);
     for (uint32_t idx = 0; idx < _mh_execute_header.ncmds; ++idx)
     {
         if (((const struct load_command *) command)->cmd == LC_ENCRYPTION_INFO_64)
         {
             struct encryption_info_command *crypt_cmd = (struct encryption_info_command *) command;
              printf("crypt_cmd--cryptid,%d\n",(uint32_t)crypt_cmd->cryptid);

              if (crypt_cmd->cryptid == 0) return false;

              if (crypt_cmd->cryptid == 1) return true;

         }
         else
         {
             command += ((const struct load_command *) command)->cmdsize;
         }
     }
     return false;
}
复制代码
```

> 兄弟们可以试一下

## 4\. 代码MD5对比

*   这个就是和我原先博客写的`代码签名原理`差不多，就是把一块重要代码块，进行`MD5比较`

*   如果有更改 --> MD5就会改变，做到防护的目的

## 5\. 总结

*   这几种方法还是比较流行的。兄得们可以在项目中用起来试试

*   希望可以和兄弟们共同探讨技术！😆，然后，小谷去加班了~


