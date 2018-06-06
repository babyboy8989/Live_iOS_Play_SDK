[TOC]
## 1.概述
Live_iOS_Push_SDK 是一个适用于iOS平台的直播SDK，使用此SDK可以与CC视频云服务进行对接，在iOS端使用CC视频的推流直播功能,本SDK只针对iOS手机端。

### 1.1 功能特性
| 功能             | 描述             |
| :--------------- | :--------------- |
| 推流直播         | 直播视频         |
| 设置码率         | 设置最大码率     |
| 设置分辨率       | 设置分辨率       |
| 设置预览页面     | 设置预览页面     |
| 设置前后置摄像头 | 设置前后置摄像头 |
| 开启闪光灯       | 开启闪光灯       |
| 设置声音大小     | 设置声音大小     |
| 聚焦到某个点     | 聚焦到某个点     |
| 设置美颜滤镜     | 设置美颜滤镜     |
| 设置水印         | 设置水印         |
| 节点测速         | 选择最优节点     |

### 1.2 阅读对象
本文档为技术文档，需要阅读者：
* 具备基本的iOS开发能力
* 准备接入CC视频的直播SDK相关功能
* 对CC云直播产品使用方法有基础的了解，[使用帮助地址](http://doc.bokecc.com/live/manual/introduction/)

## 2.开发准备

### 2.1 开发环境
* Xcode  : Xcode  开发IDE

### 2.2直播流程说明

2.2.1 登录

2.2.2 设置码率，分辨率，帧率，预览页面，前后置摄像头

2.2.3 开始推流（设置声音，闪光灯），设置推流中屏幕不会锁屏。

2.2.4 停止推流

2.2.5 退出登录（不一定退出推流就一定要退出登录，没有必然联系，但是如果退出房间，就一定要退出登录）

2.2.6 音频输入源类型有 普通耳机，蓝牙耳机，麦克风

### 2.3错误码及错误描述

以下错误都会在封装在返回的错误中的(NSError *)error中，为自定义的错误，另外reason:(NSString *)reason会是服务器返回的错误信息
1. 错误类型定义如下

typedef NS_ENUM(NSInteger, ERROR_SERVICE_TYPE) {
ERROR_PASSWORD = 1001,//@"密码错误"
ERROR_HEART_BEAT = 1002,//@"心跳请求失败"
ERROR_LONG_TIME_NOPUSH = 1003,//@"长时间未推流"
ERROR_RTMP_LINK = 1004,//@"RTMP连接失败"
ERROR_RTMP_TIMEOUT = 1005,//@"RTMP连接超时"
};

typedef NS_ENUM(NSInteger, ERROR_SYSTEM_TYPE) {
ERROR_RETURNDATA = 1006,//@"返回内容格式错误"
ERROR_PARAMETER = 1007,//@"可能是参数错误"
ERROR_NETWORK = 1008,//@"网络异常"
ERROR_NETWORK_TIMEOUT = 1009,//@"网络超时"
ERROR_ROOM_TYPE = 1010,//@"暂不支持多主讲"
};

2 错误类型分类

2.1 业务错误

1. 密码错误
2. 心跳请求失败
3. 长时间未推流
4. RTMP连接失败
5. RTMP连接超时

2.2 系统错误

1. 返回内容格式错误
2. 可能是参数错误
3. 网络异常
4. 网络超时
5. 暂不支持多主讲

## 3.快速集成

首先，需要下载最新版本的SDK，下载地址为： **[Live_iOS_Push_SDK](https://github.com/CCVideo/Live_iOS_Push_SDK)**

**集成前需要知道的事情:**
1. 因SDK中编码方式使用的是硬编码，硬编码是在iOS8.0以后才开放的，所以本SDK只适用于iOS8.0及以上版本，无法支持iOS7.x版本的系统
2. SDK在真机状态下能正常推流，在模拟器状态下不能正常推流.
3. 在Demo-推流端工程中的 CCPush.framework 和 socketio.html 直接提取出来即可
4. Info.plist配置要求
```
1.     上AppStore支持的最低版本为8.0系统
<key>LSMinimumSystemVersion</key>
<string>8.0</string>
2. 关于网络安全的配置
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>
3. 开启后台播放声音的模式
<key>UIBackgroundModes</key>
<array>
<string>audio</string>
</array>
4. 如发现
dyld: Library not loaded: @rpath/CCPush.framework/CCPush
Referenced from: /var/containers/Bundle/Application/3CC41924-7F22-48FC-AAD1-7C2C089EF749/demo.app/demo
Reason: image not found
类型的错误问题，将报错的类库添加进入
General -> Embedded Binaries 和 General -> Linked Frameworks and Libraries 两处即可
```

### 3.2 工程配置基本要求
```
1. Build Settings   ->    Build Options  ->   Enable Bitcode   ->        NO
2. General -> Embedded Binaries 添加一下CCPush.framework这个SDK
```
3.2.1**基础配置类PushParameters**
```
@property(nonatomic, copy)NSString                     *userId;//用户ID
@property(nonatomic, copy)NSString                     *roomId;//直播间号
@property(nonatomic, copy)NSString                     *viewerName;//用户名称
@property(nonatomic, copy)NSString                     *token;//密码
@property(nonatomic, assign)BOOL                       security;//是否使用https，YES:https  NO:http
```

### 3.3 登录直播间

首先导入头文件
```
#import "CCPush/CCPushUtil.h"
```
配置参数
```
PushParameters *parameters = [[PushParameters alloc] init];
parameters.userId = @"用户ID";
parameters.roomId = @"直播间号";
parameters.viewerName = @"用户名称";
parameters.token = @"密码";
parameters.security = NO;////是否使用https，YES:https  NO:http
[[CCPushUtil sharedInstanceWithDelegate:self] loginWithParameters:parameters];
```
实现代理方法
```
#pragma mark - CCPushDelegate

-(void)roomName:(NSString *)roomName {
NSLog(@"登录成功 roomName = %@",roomName);
}
/**
*    @brief    返回节点列表，节点测速时间，以及最优点索引(从0开始，如果无最优点，随机获取节点当作最优节点)
*/
- (void) nodeListDic:(NSMutableDictionary *)dic bestNodeIndex:(NSInteger)index {
//    NSLog(@"first---dic = %@,index = %ld",dic,index);
self.nodeListDic = [dic mutableCopy];

}
//@optional
/**
*    @brief    请求成功
*/
-(void)requestLoginSucceedWithViewerId:(NSString *)viewerId {
NSLog(@"登录成功 viewerId = %@",viewerId);
//跳转到设置控制器
SettingViewController *settingViewController = [[SettingViewController alloc] initWithServerDic:self.nodeListDic viewerId:viewerId roomName:@"返回的房间名称"];
[self.navigationController pushViewController:settingViewController animated:NO];
}

/**
*    @brief    登录请求失败
*/
-(void)requestLoginFailed:(NSError *)error reason:(NSString *)reason {
NSString *message = nil;
if (reason == nil) {
message = [error localizedDescription];
} else {
message = reason;
}
}
```
### 3.4 直播设置
Demo中的SettingViewController就是对直播间的一些设置,具体包含设置横竖屏,摄像头,分辨率,码率,帧率以及服务器的设置(将在网络请求类中详细介绍)
```
[[CCPushUtil sharedInstanceWithDelegate:self] startPushWithCameraFront:_isCameraFront];//开始推流
```
然后就可以开始直播了!

## 4. 网络请求类及代理介绍

CCPushUtil：推流工具类,单例类

CCPushUtilDelegate：对应的代理

### 4.1登录部分
1.登录

/**
*    @brief    服务器请求初始化
*    @param     parameters                  登陆参数
*    @return    request                     实例对象
*    @return    parameters.userId           必要参数
*    @return    parameters.roomId           必要参数
*    @return    parameters.viewerName       必要参数
*    @return    parameters.token            必要参数
*    @return    parameters.security         必要参数
*/
- (void)loginWithParameters:(PushParameters *)parameters;

2.登录对应的代理方法

/**
*    @brief    登录请求成功
*/
-(void)requestLoginSucceedWithViewerId:(NSString *)viewerId;

/**
*    @brief    登录请求失败
*/
-(void)requestLoginFailed:(NSError *)error reason:(NSString *)reason;

3.登陆成功以后到退出登陆以前可能会有用户自定义消息的代理方法被触发（定制功能，无定制需求请忽略此代理方法）

/*
* @brief    用户自定义消息
*/
- (void)customMessage:(NSString *)message;

### 4.2推流过程中会回调的代理
1.推流失败
```
/**
*    @brief    推流失败
*/
-(void)pushFailed:(NSError *)error reason:(NSString *)reason;
```
2.正在连接网络
```
/**
*    @brief    正在连接网络，注：可以进行一些UI的操作
*/
- (void) isConnectionNetWork;
```
3.连接网络完成
```
/**
*    @brief    连接网络完成
*/
- (void) connectedNetWorkFinished;
```
4.设置连接状态
```
/**
*    @brief    设置连接状态
*    status含义 1:正在连接，3:已连接(表示推流成功),5:未连接（表示推流失败）
*/
- (void) setConnectionStatus:(NSInteger)status;
```
5.点击开始推流按钮，获取liveid
```
/**
*    @brief    点击开始推流按钮，获取liveid
*/
- (void) getLiveidBeforPush:(NSString *)liveid;
```
### 4.3推流前的接口以及设置

1.最大码率
```
/**
*    @brief  得到房间可用的最大码率,注：在登录接口后，才能取到
*/
- (NSInteger)getMaxBitrate;
```
2.开始推流
```
/**
*    @brief  开始推流，注：设置好码率，分辨率，帧率，和预览页面后才可以开始推流
*   @param  cameraFront是否是前置摄像头推流
*/
- (void)startPushWithCameraFront:(BOOL)cameraFront;
```
3.设置分辨率
```
/**
*    @brief    设置分辨率videoSize，（重要：分辨率长宽一定要设置为偶数，否则推出来的视频会有绿边）建议不要设置太离谱，码率iBitRate，100到本房间可用的最大码率之间设置，
*    如果小于100，码率会设置成最小值100，如果码率大于最大码率，码率会设置成最大值，帧率iFrameRate建议设置20到30之间，
*  小于20或大于30会设置成默认值25
*/
- (void)setVideoSize:(CGSize)videoSize BitRate:(int)iBitRate FrameRate:(int)iFrameRate;
```
4.设置预览页面
```
/**
*    @brief    设置预览页面
*   @param  previewView表示预览页面，重新创建一个UIView对象，作为预览页面，
*              不要在previewView表示预览页面上面添加其他控件，也不要使用UIView的
*            子类当作参数传进去
*/
- (void)setPreview:(UIView*)previewView;
```
### 4.4推流中的接口以及设置

1. 停止推流
```
/**
*    @brief    停止推流
*/
- (void)stopPush;
```
2.停止推流的代理方法
```
/*
* @brief    停止推流成功
*/
- (void)stopPushSuccessful;
```
3.其它
```
/**
*    @brief    设置前后置摄像头
*/
- (void)setCameraFront:(Boolean)bCameraFrontFlag;

/**
*    @brief    开启闪光灯
*/
- (void)setTorch:(BOOL)torchOn;

/**
*    @brief    设置声音大小，0-10，0和10分别表示静音和最大音量
*/
- (void)setMicGain:(float)micGain;

/**
*    @brief    聚焦到某个点
*/
- (void)focuxAtPoint:(CGPoint)point ;

/**
*  @brief 设置美颜滤镜
*  @param smooth       磨皮系数 取值范围［0.0, 1.0］
*  @param white        美白系数 取值范围［0.0, 1.0］
*  @param pink         粉嫩系数 取值范围［0.0, 1.0］
*/
- (void)setCameraBeautyFilterWithSmooth:(float)smooth white:(float)white pink:(float)pink;

/**
*    @brief 设置水印(整个可设置的宽高和设置的屏幕分辨率，在切换摄像头之后需要在调用一次)
*  @param image        水印图片(去除水印的时候只需要赋值nil，再次调用该接口)
*  @param rect         坐标 取值范围(设置的分辨率的宽高)
*/
- (void)addWaterMask:(UIImage *)image rect:(CGRect)rect;
```
### 4.5没有推流时可以请求的接口以及设置(此接口和推流无关)

/**
*    @brief    退出登录
*/
- (void)logout;

/**
*    @brief    单例并设置代理
*/
+ (instancetype)sharedInstanceWithDelegate:(id)delegate;

### 4.6 测速

说明:对我们机房节点进行测速，无意外情况下，应该是离推流地点最近以及网速最好两者结合的最优节点

4.6.1 测速代理方法

说明：在登录时会自动进行一次测速,测速结果通过以下代理通知给用户：
/**
*    @brief    返回节点列表，节点测速时间，以及最优点索引
*   (从0开始，如果无最优点，随机获取节点当作最优节点)
*/
- (void) nodeListDic:(NSMutableDictionary*)dic bestNodeIndex:(NSInteger)index;

注1：字典中key为节点名称：value为一个字典，字典中有key:@"time"表示测速时间(ms)和key:@"index"表示节点索引

注2：筛选的节点有一个名称为全球节点，当其他节点推流困难或推不上流时，可用这个全球节点推流，全球节点不进行测速，故没有测速结果

4.6.2 手动测速

说明：在登录以后，即第一次测速完成，且未开始推流的时候调用CCPushUtil类中的以下方法可以重新测速,每次测速完成后都会调用3.6.1中的代理方法返回测速结果：
/**
*    @brief    测速
*/
- (void) testSpeed;

4.6.3 手动设置推流到的服务器节点

说明：测速以后，会给出一个最优节点(或随机节点)，如果不调用以下接口，会使用最优节点(或随机节点)进行推流，如果调用以下接口，会根据手动设置的节点推流到推流服务器，设置的时间点一定要在测速完成后，推流开始前，索引值为从0到测速节点减1之间的数字，不能越界，否则会崩溃：
/**
*    @brief    设置推流节点索引，如果不设置，默认首选测速最优点(如果测速结果全部超时则会选择随机点)
*    @param     index    节点索引
*/
- (void) setNodeIndex:(NSInteger)index;

### 4.7 聊天的代理（如果不对接聊天功能，请忽略）

/**
*    @brief    收到私聊信息
*/
- (void)on_private_chat:(NSString *)str;

/**
*    @brief  收到公聊信息
*/
- (void)on_chat_message:(NSString *)str;

### 4.8  获取房间人数以及房间信息的代理方法

/**
*    @brief    获取当前在线人数
*  调用- (void)roomUserCount;方法会触发此代理
*/
- (void)room_user_count:(NSString *)str; 

/**
*    @brief    获取房间用户列表
*  调用- (void)roomContext;方法会触发此代理
*/
- (void)receivePublisherId:(NSString *)str onlineUsers:(NSMutableDictionary *)dict;

### 4.9 聊天调用方法（如果不对接聊天功能，请忽略）

/**
*    @brief  发送公聊信息
*/
- (void)chatMessage:(NSString *)message ;

/**
*    @brief  发送私聊信息
*/
- (void)privateChatWithTouserid:(NSString *)touserid msg:(NSString *)msg;

### 4.10 获取房间人数以及房间信息

/**
*    @brief  询问房间信息，同上面接口相似的是，此接口一定要在登录成功后才可以调用，登录不成功或者退出登录后不能调用此接口，因为此房间信息中包含用户列表信息，所以可以采用定时器循环调用的方法来调用，也可以在有用户，黑名单，白名单信息变动的时候调用，在SDK里面和此信息紧紧相关联的是聊天信息，如果有用户变动，没有及时调用此接口的话，那在调用SDK聊天系统的时候，可能会因为取不到发消息的用户的信息而产生崩溃，如果不使用推流SDK的聊天系统的话，可以忽略此接口
此方法对应代理：- (void)receivePublisherId:(NSString *)str onlineUsers:(NSMutableDictionary *)dict;
*/
- (void)roomContext;

/**
*    @brief  获取在线房间人数，当登录成功后即可调用此接口，推不推流都能够调用，并且都会有返回值，登录不成功或者退出登录后就不可以调用了，如果要求实时性比较强的话，可以写一个定时器，不断调用此接口，几秒中发一次就可以，然后在代理回调函数中，处理返回的数据
此方法对应代理：- (void)room_user_count:(NSString *)str;
*/
- (void)roomUserCount;

### 4.11 添加或清除直播时的贴图

/**
* @brief 添加直播图片(添加图片到直播视频中，把图片直播出去)
* @param image 图片
* @param isBig 图片模式(大图、小图)，如果isBig=YES是把传进去的图片作为背景，视频缩小然后铺在图片的一角，     如果isBig=NO是把传进去的图片铺在视频上面的一角
*/
- (void)publishImage: (UIImage*) image isBig:(BOOL)isBig;

/**
* @brief 清除直播图片
*/
- (void)clearPublishImage;

## 5.API查询

Doc目录打开index.html文件

## 6.Q&A
### 6.1如发现 dyld: Library not loaded: @rpath/CCPush.framework/CCPush
Referenced from: /var/containers/Bundle/Application/3CC41924-7F22-48FC-AAD1-7C2C089EF749/demo.app/demo
Reason: image not found问题
将报错的类库添加进入
General -> Embedded Binaries 和 General -> Linked Frameworks and Libraries 两处即可

