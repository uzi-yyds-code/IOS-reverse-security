

# 更新最新热门文章

*   [逆向入门到进阶项目合集](https://github.com/uzi-yyds-code/IOS-reverse-security)



* **欢迎可以加入iOS高级技术交流群：[1001906160](https://jq.qq.com/?_wv=1027&k=KjioxJty)；群文件可以免费真机包下载，更多逆向视频、中高级进阶资料！**

## 1\. 引言


*   小谷的公司每个月可以迟到3次，小谷5月的时候考勤竟然迟到了4次。很难受。为了杜绝这种情况。毅然决然的研究一波`偷偷打卡😃`

*   小谷用的设备是`iPhone XS Max`，`iOS 14.3（越狱`），目前`最新的钉钉：6.0.16`。

* iOS 14.3设备下载：https://github.com/uzi-yyds-code/IOS-device-debugging-package


*   用`Windows`做`逆向`的兄弟们，经常使用`IDA和Reveal`还有`终端debugserver附加进程`，今天小谷要用iOS开发的兄弟们都比较喜欢的`Xcode和hopper`。😆（其实都差不多。）

*   先看效果图

![0.png](https://upload-images.jianshu.io/upload_images/19704571-4e0435d49f21c411.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2\. 钉钉打卡插件

### 2.1\. 调试应用

> 蹂躏第一步，首先要装酷

*   先把钉钉砸壳，取下来。我使用的是`frida-ios-dump`,小谷写过一篇[砸壳和Frida](https://juejin.cn/post/6966504711935655950/)的博客.

![1.1.png](https://upload-images.jianshu.io/upload_images/19704571-79b51fb2e9b83cc5.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![1.2.png](https://upload-images.jianshu.io/upload_images/19704571-a526fd8d0652393b.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这样`砸过壳`的 `钉钉.ipa`就出来了，先放这，我们先不管他

*   开始了！ `iOS`兄弟们超级喜欢的，`Xcode`调试（曾经有兄弟问过我怎么附加的。我这里画下流程图）

![2.png](https://upload-images.jianshu.io/upload_images/19704571-2a79d04f93d0ea8e.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 这样就清晰很多了

*   既然我们可以调试的话，那就直接`定位打卡`的代码了啊 （找到打卡界面，`viewdebug`调试）

![3.png](https://upload-images.jianshu.io/upload_images/19704571-75e93c793697540f.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我滴天啊。是个`WKWebview`,定位不到!!

`这样就结束了吗？怎么可能呢，那以后小谷要是迟到，没有办法弥补了~·`

#### 2.1.1\. 思路

> 整理下思路和线索

*   线索：
    *   打卡界面是个`WKWebview`界面
    *   我们打卡成功的时候`H5`会有变化，说明是有地方`告诉H5`定位成功
    *   如果是原生定位`告诉H5`的，那么他们就一定有`交互代码`
*   思路:
    *   如果他们有`交互`，我们知道`hook住`这个`交互`就可以了(看里面的`回传`，通过改变值可能做到)
    *   如果是iOS的定位，一定会走 `locationManager: didUpdateLocations:`
    *   很有可能就是，`H5交互的时候开始定位`，然后`回调定位信息`

> 那么我们接下来的任务就是`定位交互的代码`和`打印回传的信息`了！！ 搞起，搞起~

### 2.2\. 定位代码

*   我们刚开始导出了`钉钉.ipa`,`解压` ，拿`MachO`文件

*   然后通过`class-dump`取出`header`

`class-dump -H DingTalk -o dingHeader`

*   把二进制文件拖进`Hopper` (搜索`locationManager: didUpdateLocations:`)

![4.png](https://upload-images.jianshu.io/upload_images/19704571-8e73af39eac41028.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 一共就`5`个，我们就把这几个`hook`一下，看下`log`了，我猜测应该就可以定位到了(如果他调用的`原生的`)

*   这次我们就用`Monkey`写`Tweak插件`了(当然也可以用`THEOS`，主要我们这次可能要好几次调试才能定位，我就用`Monkey`方便点~)

![5.png](https://upload-images.jianshu.io/upload_images/19704571-a4846df234309b81.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   获取`钉钉的APPID`,并配置

![6.png](https://upload-images.jianshu.io/upload_images/19704571-fb7d9ea5b9a44e14.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![7.png](https://upload-images.jianshu.io/upload_images/19704571-d0454a42ebc198c1.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   我们把上面找到的那`5`个，`hook`一下

```
#import <UIKit/UIKit.h>

%hook AMapLocationCLMDelegate
- (void)locationManager:(id)arg1 didUpdateLocations:(id)arg2{
    %log;
    %orig;
}
%end

%hook AMapLocationManager
- (void)locationManager:(id)arg1 didUpdateLocations:(id)arg2{
    %log;
    %orig;
}
%end

%hook LALocationManager
- (void)locationManager:(id)arg1 didUpdateLocations:(id)arg2{
    %log;
    %orig;
}
%end

%hook DTCLocationManager
- (void)locationManager:(id)arg1 didUpdateLocations:(id)arg2{
    %log;
    %orig;
}
%end

%hook MAMapView
- (void)locationManager:(id)arg1 didUpdateLocations:(id)arg2{
    %log;
    %orig;
}
%end
复制代码
```

> 然后我们安装插件，看下`log`

*   我们发现这个`AMapLocationCLMDelegate`和`AMapLocationManager`都是成对出现的，其他的都没有走

![8.png](https://upload-images.jianshu.io/upload_images/19704571-6ad7ad566d756017.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   然后我们观察下`class-dump`出来的`header`文件。发现`AMapLocationCLMDelegate`里面都是一些代理回调

> 这个时候，小谷又可以推测一波了： 在`AMapLocationManager`获取的位置，然后参数在把参数`传给H5交互`

*   我们把`AMapLocationManager`里面的方法都打一遍`log`,使用`logify.pl`

![9.png](https://upload-images.jianshu.io/upload_images/19704571-49150d1cf042bc9d.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   然后我们在次装上插件看下`log`

![11.png](https://upload-images.jianshu.io/upload_images/19704571-0fe7bfa2638aa979.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 看到了不错的信息。我们要继续开始`Xcode调试附加`了

*   从`Hopper`找到偏移位置

![12.png](https://upload-images.jianshu.io/upload_images/19704571-de96d5fe7df9ba33.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   `Xcode`附加下，可以找到`ASLR`

![13.png](https://upload-images.jianshu.io/upload_images/19704571-f300b3191ac4b2ac.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   定位偏移，设置断点

![14.png](https://upload-images.jianshu.io/upload_images/19704571-3f8de9427cd17acc.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> `0x104e70000 + 0x36b4a8 = 0x1051DB4A8`

*   当我点击考勤打卡的时候，断住了~

![15.png](https://upload-images.jianshu.io/upload_images/19704571-a447ee1fd956cc34.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   根据我们的思路，我们要看`函数调用栈`了！！

![16.png](https://upload-images.jianshu.io/upload_images/19704571-fa9da8addacf88c9.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 有没有那么一丢丢爽歪歪~ (如果没有符号也没有关系，我们就通过`Hopper`地址定位！)

*   我们`hook`下这个函数

```
%hook LAPluginInstanceCollector
- (void)handleJavaScriptRequest:(id)arg1 callback:(id)arg2{
    %log;
    %orig;
}
%end
复制代码
```

*   兄弟们~ 我们可以得到线索，这个`callback`是个`block`，说明`arg1`是参数，`arg2`是回调回去的参数

![17.png](https://upload-images.jianshu.io/upload_images/19704571-7fbde2c78ab1b292.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   想知道这个`block`的参数类型。就要看看他的`签名`了~

![18.png](https://upload-images.jianshu.io/upload_images/19704571-32b1b08d926b5cf6.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 通过`内存平移`来拿他的`签名`~

*   老办法，`Hopper`拿地址。然后`Xcode附加下断点`

![19.png](https://upload-images.jianshu.io/upload_images/19704571-60d883b4cbca05fc.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ![20.png](https://upload-images.jianshu.io/upload_images/19704571-6ee5d060960269d3.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> `0x100f50000 + 0x488f110 = 0x1057DF110`

*   `b 0x1057DF110` 断住它,然后分析他

![21.png](https://upload-images.jianshu.io/upload_images/19704571-adfbb09bb765806a.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ![22.png](https://upload-images.jianshu.io/upload_images/19704571-5c4e6a4dc180648a.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   这个时候们就可以继续`hook`了，看看他`里面的参数`是啥

```
%hook LAPluginInstanceCollector

- (void)handleJavaScriptRequest:(id)arg1 callback:(void (^) (id))arg2{

    //我们可以自定义个block看下。
    id xg_callback = ^(id argCb){
        //可以先把传入的参数打印下，看看有没有啥触发器
        NSLog(@"xg_callback arg1:%@",arg1);
        //然后把传入的参数打印下,也看下他是什么类型的
        NSLog(@"xg_callback argCbClass:%@, argCb:%@",[argCb class],argCb);
        arg2(argCb);
    };
    //xg_callback替换arg2。（只做了一个转接的过程）
    %orig(arg1,xg_callback);
}
%end
复制代码
```

*   然后看效果

![23.png](https://upload-images.jianshu.io/upload_images/19704571-8fee405527886120.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 我好像找到了，但是这里面还有好多其他跟他相关连的信息。（比如地址，城市，经纬度啥的)

### 2.3\. 打卡插件代码

> 我们知道他类型是个字典。`action=start`的时候触发

*   直接上代码了（最新版）

```
%hook LAPluginInstanceCollector
- (void)handleJavaScriptRequest:(id)arg1 callback:(void (^) (id))arg2{
    id xg_callback = ^(id argCb){
        //小谷做的时候其实打了好多log。。
        NSDictionary *arg1Dic = (NSDictionary *)arg1;
        if([arg1Dic[@"action"] isEqualToString:@"start"]){
            NSLog(@"xg_callback text:start");
            NSMutableDictionary * CbDic = [NSMutableDictionary dictionaryWithDictionary:argCb];
            if (CbDic[@"result"][@"latitude"] && CbDic[@"result"][@"longitude"]){
                NSLog(@"xg_callback text:result");
                NSMutableDictionary * resultDic = [NSMutableDictionary dictionaryWithDictionary:CbDic[@"result"]];
                resultDic[@"latitude"] = @"40.0361208767361";
                resultDic[@"longitude"] = @"116.4161067708333";
                [CbDic setValue:resultDic forKey:@"result"];
            }
            arg2(CbDic);
        }else{
            arg2(argCb);
        }
    };
    //xg_callback替换arg2。（只做了一个转接的过程）
    %orig(arg1,xg_callback);
}
%end
复制代码
```

*   看下效果图

![24.png](https://upload-images.jianshu.io/upload_images/19704571-55a8c1cb712d9319.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 3\. 设置打卡开关

> 主要功能写的差不多了。我们再来设置一个开关来控制一下：如果`开关开启`，就走`打卡插件逻辑`，如果`开关关闭`，走`原来的逻辑`。

*   我们在设置里面偷偷的加个按钮~

![25.png](https://upload-images.jianshu.io/upload_images/19704571-3965db435cfe320b.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> `Xcode`好强大~

*   然后找下`dataSource`吧

![26.png](https://upload-images.jianshu.io/upload_images/19704571-14dfe71ee0b8c529.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   那这就好办了啊~ （兄得们，这是个`tableview`啊，把他的代码一`hook` 加行`cell`，不是很`easy`吗）

```
@interface DTTableViewHandler : NSObject
- (long long)numberOfSectionsInTableView:(UITableView *)tableView;
@end

%hook DTTableViewHandler

%new
-(void)xg_switchChang:(UISwitch *)switchView{
    [XGDefaults setBool:switchView.isOn forKey:XGSWITCHKEY];
    [XGDefaults synchronize];
}

- (long long)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
//    如果是最后一组，增加一行
    if([tableView.nextResponder .nextResponder isKindOfClass:%c(DTSettingListViewController)] && (section == [self numberOfSectionsInTableView:tableView]-1)){
        return 1;
    }
    return %orig;
}

- (id)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    //最后一组，最后一行，设置一个cell
    if([tableView.nextResponder .nextResponder isKindOfClass:%c(DTSettingListViewController)] && ([indexPath section] == [self numberOfSectionsInTableView:tableView]-1)){
        UITableViewCell * cell = nil;
        if([indexPath row] == 0){
            static NSString * swCell = @"SWCELL";
            cell = [tableView dequeueReusableCellWithIdentifier:swCell];
            if(!cell){
                cell = [[UITableViewCell alloc] initWithStyle:(UITableViewCellStyleDefault) reuseIdentifier:nil];
            }
            cell.textLabel.text = @"偷偷打卡开启！！";
            UISwitch * switchView = [[UISwitch alloc] init];
            switchView.on = [XGDefaults boolForKey:XGSWITCHKEY];
            [switchView addTarget:self action:@selector(xg_switchChang:) forControlEvents:(UIControlEventValueChanged)];
            cell.accessoryView = switchView;
            cell.backgroundColor = [UIColor whiteColor];
            return cell;
        }
        return nil;
    }else{
        return %orig;
    }
}

- (double)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    //设置最后一组最后一行的行高
    if([tableView.nextResponder .nextResponder isKindOfClass:%c(DTSettingListViewController)] && ([indexPath section] == [self numberOfSectionsInTableView:tableView]-1)){
        return 45;
    }
    return %orig;
}

- (long long)numberOfSectionsInTableView:(UITableView *)tableView {
    //增加一组
    if([tableView.nextResponder .nextResponder isKindOfClass:%c(DTSettingListViewController)]){
        return %orig + 1;
    }
    return %orig;
}
%end
复制代码
```

*   然后在原来打卡的地方加个判断，就好了

`if([arg1Dic[@"action"] isEqualToString:@"start"] && [XGDefaults boolForKey:XGSWITCHKEY]){`

*   看效果

![0.png](https://upload-images.jianshu.io/upload_images/19704571-d0ce9257dee12445.image?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 更新最新热门文章

*   [逆向入门到进阶项目合集](https://github.com/uzi-yyds-code/IOS-reverse-security)



* **欢迎可以加入iOS高级技术交流群：[1001906160](https://jq.qq.com/?_wv=1027&k=KjioxJty)；群文件可以免费真机包下载！**




作者：小谷先森
链接：https://juejin.cn/post/6969220825455493133

