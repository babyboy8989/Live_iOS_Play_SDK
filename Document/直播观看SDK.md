[TOC]
## 1.概述
Live_iOS_Play_SDK 是一个适用于iOS平台的直播SDK，使用此SDK可以与CC视频云服务进行对接，在iOS端使用CC视频的直播功能,本SDK只针对iOS手机播放端。

### 1.1 功能特性
| 功能     | 描述                           |
| :------- | :----------------------------- |
| 直播视频 | 观看直播视频                   |
| 文档展示 | 能够观看当前直播文档           |
| 线路更换 | 观看卡顿请换个线路             |
| 清晰度   | 支持直播多清晰度切换播放       |
| 答题卡   | 支持实时检测课堂学生的掌握程度 |
| 问答     | 能够发送问题和接受回答信息     |
| 简介     | 支持对直播间的信息展示         |
| 问卷     | 支持对观看直播的人进行信息采集 |
| 广播     | 支持发送全体消息               |
| 连麦     | 支持与直播人员进行音视频沟通   |
| 签到     | 支持签到功能                   |
| 抽奖     | 支持抽奖功能                   |
| 投票     | 支持投票功能                   |



### 1.2 阅读对象
本文档为技术文档，需要阅读者：
* 具备基本的iOS开发能力
* 准备接入CC视频的直播SDK相关功能
* 对CC云直播产品使用方法有基础的了解，[使用帮助地址](http://doc.bokecc.com/live/manual/introduction/)
## 2.开发准备

### 2.1 开发环境
* Xcode  : Xcode  开发IDE
### 2.2错误码及错误描述
(1). 业务错误

typedef NS_ENUM(NSInteger, ERROR_SERVICE_TYPE) {
ERROR_ROOM_STATE = 1001,//@"直播间状态不可用，可能没有开始推流"
ERROR_USELESS_INFO = 1002,//@"没有获取到有用的视频信息"
ERROR_PASSWORD = 1003,//@"密码错误"
};

(2). 系统错误

typedef NS_ENUM(NSInteger, ERROR_SYSTEM_TYPE) {
ERROR_RETURNDATA = 1004,//@"返回内容格式错误"
ERROR_PARAMETER = 1005,//@"可能是参数错误"
ERROR_NETWORK = 1006,//@"网络错误"
};

**日志工具类**
(1) 单例模式

/**
*    @brief  获取单例对象
*/
+ (instancetype)sharedInstance;

/**
*    @brief    是否需要存储日志信息到文件中，最好在AppDelegate类里面调用，这样就适用于全局日
*           志的存储，当然中途调用也可以，从调用后开始应用之后的日志存储逻辑
*/
-(void)isNeedToSaveLog:(BOOL)needsave;

/**
*    @brief    存储日志的信息
*           action相当于key，也可以理解为存储的事件名称
*            logStr相当于value，也可以理解为存储的日志信息字符串
*/
-(void)saveLog:(NSString *)logStr action:(NSString *)action;

(2) 示例应用及说明

在CCSDK中相关的信息已经做了日志打印的相关处理操作，但是在demo中也添加了几句话，用户也可以自定义添加自己需要的日志，用于记录或者调试都可以。

注意：

1) SDK中设置的日志存储目录为：NSHomeDirectory()/Library/CCLog/  ，存储文件名为yyyy-MM-dd.log

2) SDK中如果开启了日志存储以后，日志只存不删，需要使用者按照自己的逻辑进行删除

1. 在AppDelegate.m中添加了
[[SaveLogUtil sharedInstance]isNeedToSaveLog:YES];
表示设置为记录日志log

2. 在PlayForPCVC.m中添加了
[[SaveLogUtil sharedInstance] saveLog:@"" action:@"视频加载成功或开始播放，多次调用，不必关心"];
表示视频加载成功或开始播放

3. PlayBackVC.m中添加了
[[SaveLogUtil sharedInstance] saveLog:@"" action:@"视频加载成功或开始播放，多次调用，不必关心"];
表示视频加载成功或开始播放

## 3. 快速集成

首先，需要下载最新版本的SDK，下载地址为：[Live_iOS_Play_SDK](https://github.com/CCVideo/Live_iOS_Play_SDK)
集成前需要知道的事情:
1. 工程分三个部分：一个有连麦功能（Demo-观看端有连麦），体积较大，一个无连麦功能（Demo-观看端无连麦），体积较小，第三个部分是离线下载回放（Demo-离线下载），其中离线下载回放中的CCSDK用有连麦的库和无连麦的库都可以，示例中用的无连麦的库(体积较小)
2. （重要）本SDK只支持真机，不支持模拟器
3. （重要）本SDK只支持iOS8.0及以上版本，iOS7.x版本不支持
4. (重要) 当使用有连麦工程时，分为音视频连麦和音频连麦两种连麦，有连麦的demo中没有以页面体现两种连麦的按钮，当选中简介时，连麦为音视频连麦，当选中 文档，聊天或问答任意一个时，连麦为音频连麦，代码使中用_isAudioVideo加以区分，当_isAudioVideo==YES;时为音视频连麦，当_isAudioVideo==NO;时为音频连麦
### 3.1 导入framework
三个工程SDK目录都在工程目录下的CCSDK文件夹中，文件夹中的所有文件都属于SDK部分，应全部应用(注意不要遗漏socketio.html文件)

### 3.2 Demo
1. Demo-观看端有连麦实现了基本的播放，在线回放逻辑，可以直接运行
2. Demo-观看端无连麦实现了基本的播放，在线回放逻辑，可以直接运行，除无连麦外，功能同上
3. Demo-离线下载实现了基本的离线下载和离线回放逻辑，可以直接运行（其中的CCSDK用4，5两个工程中的任意一个CCSDK文件夹即可）

### 3.3 配置依赖库

在项目/General/Linked framework and libraries 中不要忘记添加对用的依赖
**基础配置类PlayParameter**
```
@property(nonatomic, copy)NSString                      *userId;//用户ID
@property(nonatomic, copy)NSString                      *roomId;//房间ID
@property(nonatomic, copy)NSString                      *viewerName;//用户名称
@property(nonatomic, copy)NSString                      *token;//房间密码
@property(nonatomic, copy)NSString                      *liveId;//回放ID，回放时才用到
@property(nonatomic, copy)NSString                      *recordId;//回放ID
@property(nonatomic, copy)NSString                      *viewerCustomua;//用户自定义参数，需和后台协商，没有定制传@""
@property(nonatomic, copy)NSString                      *destination;//下载文件解压到的目录路径(离线下载相关)
@property(nonatomic,strong)UIView                       *docParent;//文档父类窗口
@property(nonatomic,assign)CGRect                       docFrame;//文档区域
@property(nonatomic,strong)UIView                       *playerParent;//视频父类窗口
@property(nonatomic,assign)CGRect                       playerFrame;//视频区域
@property(nonatomic,assign)BOOL                         security;//是否使用https，静态库暂时只能使用http协议
/*
* 0:IJKMPMovieScalingModeNone
* 1:IJKMPMovieScalingModeAspectFit
* 2:IJKMPMovieScalingModeAspectFill
* 3:IJKMPMovieScalingModeFill
*/
@property(assign, nonatomic)NSInteger                   scalingMode;//屏幕适配方式，含义见上面
@property(nonatomic,strong)UIColor                      *defaultColor;//ppt默认底色，不设置(nil)默认为白色
@property(nonatomic,assign)BOOL                         pauseInBackGround;//后台是否继续播放，注意：如果开启后台播放需要打开 xcode->Capabilities->Background Modes->on->Audio,AirPlay,and Picture in Picture
/*
* PPT适配模式分为三种，
* 1.一种是全部填充屏幕，可拉伸变形，
* 2.第二种是等比缩放，横向或竖向贴住边缘，另一方向可以留黑边，
* 3.第三种是等比缩放，横向或竖向贴住边缘，另一方向出边界，裁剪PPT，不可以留黑边
*/
@property(assign, nonatomic)NSInteger                   PPTScalingMode;//PPT适配方式，含义见上面
```

### 3.4 登录直播间
**可以登录也可以不登录,本文档是按照需要登录设计的,如不需登录请跳过3.4**
(1). 首先导入头文件
```
#import "CCSDK/CCLiveUtil.h"
#import "CCSDK/RequestData.h"
```
(2). 配置参数
```
PlayParameter *parameter = [[PlayParameter alloc] init];
parameter.userId = @"用户ID";
parameter.roomId = @"房间ID";
parameter.viewerName = @"用户名称";
parameter.token = @"房间密码";
parameter.security = @"是否使用https，静态库暂时只能使用http协议";
parameter.viewerCustomua = @"viewercustomua";
RequestData *requestData = [[RequestData alloc] initLoginWithParameter:parameter];
requestData.delegate = self;//需要实现RequestDataDelegate代理方法
```
(3). 实现代理方法
```
#pragma mark - RequestDataDelegate
//@optional
/**
*    @brief    请求成功
*/
-(void)loginSucceedPlayBack {
控制器跳转
}

/**
*    @brief    登录请求失败
*/
-(void)loginFailed:(NSError *)error reason:(NSString *)reason {
NSString *message = nil;
if (reason == nil) {
message = [error localizedDescription];
} else {
message = reason;
}
}
```
### 3.5 开启观看视频直播
跳转到直播页面配置观看直播同样需要导入头文件
```
#import "CCSDK/CCLiveUtil.h"
#import "CCSDK/RequestData.h"
```
配置参数,和登录时配置的参数差不多,很多参数都是登录传过来的
```
PlayParameter *parameter = [[PlayParameter alloc] init];
parameter.userId = @"登录传过来的";
parameter.roomId = @"登录传过来的";
parameter.viewerName = @"用户名称";
parameter.token = @"房间密码";
parameter.playerParent = self.videoView;//视频父类窗口
parameter.playerFrame = _videoRect//视频区域;
parameter.security = NO;
parameter.PPTScalingMode = 2;
parameter.defaultColor = [UIColor whiteColor];
parameter.scalingMode = 1;
parameter.pauseInBackGround = NO;
parameter.viewerCustomua = @"viewercustomua";
_requestData = [[RequestData alloc] initWithParameter:parameter];

```

### 3.6 文档观看
PlayParameter增加两个属性
```
@property(nonatomic,strong)UIView                       *docParent;//文档父类窗口
@property(nonatomic,assign)CGRect                       docFrame;//文档区域
```
## 4. 功能使用
### 4.1  RequestData直播请求类介绍
#### 4.1.1 在直播类PlayViewController.m文件中主要调用此类来和此SDK交互

#### 4.1.2 RequestData中的成员变量
```
@property (weak,nonatomic) id<RequestDataDelegate>      delegate;//代理
@property (retain,    atomic) id<IJKMediaPlayback>      ijkPlayer;//播放器
```
#### 4.1.3 RequestData中的成员方法

(1). 初始化登录
```
/**
*    @brief    登录房间
*    @param     parameter   配置参数信息
*  必填参数 userId;
*  必填参数 roomId;
*  必填参数 viewerName;
*  必填参数 token;
*  必填参数 security;
*  （选填参数） viewercustomua;
*/
- (id)initLoginWithParameter:(PlayParameter *)parameter;
```
(2). 进入房间
```
/**
*    @brief    进入房间，并请求画图聊天数据并播放视频（可以不登陆，直接从此接口进入直播间）
*    @param     parameter   配置参数信息
*  必填参数 userId;
*  必填参数 roomId;
*  必填参数 viewerName;
*  必填参数 token;
*  必填参数 docParent;
*  必填参数 docFrame;
*  必填参数 playerParent;
*  必填参数 playerFrame;
*  必填参数 PPTScalingMode;
*  必填参数 security;
*  必填参数 defaultColor;
*  必填参数 scalingMode;
*  必填参数 pauseInBackGround;
*  （选填参数） viewercustomua;
*/
- (id)initWithParameter:(PlayParameter *)parameter;
```
(3). 提问
```
/**
*    @brief    提问
*    @param     message 提问内容
*/
- (void)question:(NSString *)message;
```
(4). 发送公聊
```
/**
*    @brief    发送公聊信息
*    @param     message  发送的消息内容
*/
- (void)chatMessage:(NSString *)message;
```
(5). 发送私聊
```
/**
*    @brief  发送私聊信息
*/
- (void)privateChatWithTouserid:(NSString *)touserid msg:(NSString *)msg;
```
(6). 退出播放时销毁文档和视频
```
/**
*    @brief    销毁文档和视频，清除视频和文档的时候需要调用,推出播放页面的时候也需要调用
*/
- (void)requestCancel;
```
(7). 房间人数
```
/**
*    @brief  获取在线房间人数，当登录成功后即可调用此接口，登录不成功或者退出登录后就不可以调用了，如果要求实时性比较强的话，可以写一个定时器，不断调用此接口，几秒钟发一次就可以，然后在代理回调函数中，处理返回的数据
*/
- (void)roomUserCount;
```
(8). 文档宽高比
```
/**
*    @brief  获取文档区域内白板或者文档本身的宽高比，返回值即为宽高比，做屏幕适配用
*/
- (CGFloat)getDocAspectRatio;
```
(9). 改变文档区域大小
```
/**
*    @brief  改变文档区域大小,主要用在文档生成后改变文档窗口的frame
*/
- (void)changeDocFrame:(CGRect) docFrame;
```
(10). 改变播放器frame
```
/**
*    @brief  改变播放器frame
*/
- (void)changePlayerFrame:(CGRect) playerFrame;
```
(11). 改变播放器父窗口
```
/**
*    @brief  改变播放器父窗口
*/
- (void)changePlayerParent:(UIView *) playerParent;
```
(12). 改变文档父窗口
```
/**
*    @brief  改变文档父窗口
*/
- (void)changeDocParent:(UIView *) docParent;
```
(13). 播放器暂停
```
/**
*    @brief  播放器暂停
*/
- (void)pausePlayer;
```
(14). 播放器播放
```
/**
*    @brief  播放器播放
*/
- (void)startPlayer;
```
(15). 播放器关闭并移除
```
/**
*    @brief  播放器关闭并移除
*/
- (void)shutdownPlayer;
```
(16). 播放器停止
```
/**
*    @brief  播放器停止
*/
- (void)stopPlayer;
```
(17). 切换播放线路
```
/**
*    @brief   切换播放线路
*  firIndex表示第几个源
*  key表示该源对应的描述信息
*/
- (void)switchToPlayUrlWithFirIndex:(NSInteger)firIndex key:(NSString *)key;
```
(18). 重新加载视频
```
/*
* 重新加载视频,参数force表示是否强制重新加载视频，
* 一般重新加载视频的时间间隔应该超过3秒，如果强制重新加载视频，时间间隔可以在3S之内
*/
-(void)reloadVideo:(BOOL)force;
```
(19). 签到
```
/*
*签到
*/
-(void)answer_rollcall;
```
(20). 答单选题
```
/*
*答单选题
*/
-(void)reply_vote_single:(NSInteger)index;
```
(21). 答多选题
```
/*
*答多选题
*/
-(void)reply_vote_multiple:(NSMutableArray *)indexArray;
```
(22). 播放器是否播放
```
/**
*    @brief  播放器是否播放
*/
- (BOOL)isPlaying;
```
(23). 设置后台是否可播放
```
/**
*    @brief  设置后台是否可播放
*/
- (void)setpauseInBackGround:(BOOL)pauseInBackGround;
```
(24). 提交问卷结果
```
/*
*  @brief 提交问卷结果
*/
-(void)commitQuestionnaire:(NSDictionary *)dic;
```
(25). 连麦相关方法
```
/*(连麦相关方法)
* 当收到- (void)acceptSpeak:(NSDictionary *)dict;回调方法后，调用此方法
* dict 正是- (void)acceptSpeak:(NSDictionary *)dict;接收到的的参数
* remoteView 是远程连麦页面的view，需要自己设置并发给SDK，SDK将要在这个view上进行远程画面渲染
*/
- (void)saveUserInfo:(NSDictionary *)dict remoteView:(UIView *)remoteView;
```
```
/*(连麦相关方法)
* 观看端主动断开连麦时候需要调用的接口
*/
- (void)disConnectSpeak;
```
```
/*(连麦相关方法)
* 当观看端主动申请连麦时，需要调用这个接口，并把本地连麦预览窗口传给SDK，SDK会在这个view上
* 进行远程画面渲染
*/
-(void)requestAVMessageWithLocalView:(UIView *)localView;
```
```
/*(连麦相关方法)
*设置本地预览窗口的大小，连麦成功后调用才生效，连麦不成功调用不生效
*/
-(void)setLocalVideoFrameA:(CGRect)localVideoFrame;
```
```
/*(连麦相关方法)
*设置远程连麦窗口的大小，连麦成功后调用才生效，连麦不成功调用不生效
*/
-(void)setRemoteVideoFrameA:(CGRect)remoteVideoFrame;
```
```
/*(连麦相关方法)
*将要连接WebRTC
*/
-(void)gotoConnectWebRTC;
```
**连麦方法调用顺序**
```
（1）-(void)gotoConnectWebRTC; //申请视频连麦

（2）回调- (void)whetherOrNotConnectWebRTCNow:(BOOL)connect;//现在情况下是否可以连麦

（3）-(void)requestAVMessageWithLocalView:(UIView *)localView;//连麦，传进去本机的预览窗口view

（4）回调- (void)acceptSpeak:(NSDictionary *)dict;//主播接受视频连麦

（5）- (void)saveUserInfo:(NSDictionary *)dict remoteView:(UIView *)remoteView;//传回dict数据，和远端的视图view

（6）回调- (void)connectWebRTCSuccess;//表示连麦成功，不成功不用管，会自动挂断

断开连麦：

（1）- (void)disConnectSpeak;//主动断开连接

（2）回调-(void)speak_disconnect:(BOOL)isAllow;//参数isAllow表示是否直播间允许连麦，在这里写一些UI的操作，一般里面都会调用- (void)allowSpeakInteraction:(BOOL)isAllow;这个方法类似的内容

（3）回调方法- (void)allowSpeakInteraction:(BOOL)isAllow;//表示此房间是否允许连麦，一般做一些UI的内容，此方法不止调用一次（登陆，断网联网，改变房间是否允许连麦状态的时候都会回调此代理方法），尽量不要做一些重复的操作

连麦预览中可以调用：

（1）-(void)setLocalVideoFrameA:(CGRect)localVideoFrame;//修改本地预览窗口大小，连麦过程中，本地预览窗口就销毁，因为不会展示本地预览窗口了，只会展示远端窗口

连麦过程中调用：

（1）-(void)setRemoteVideoFrameA:(CGRect)remoteVideoFrame;//修改远端视频窗口的大小，推流中只会展示远端窗口

```

### 4.2 事件监听

事件监听只需要实现对应的代理方法即可


#### 4.2.1 请求播放地址成功
```
/**
*    @brief    请求播放地址成功
*/
-(void)requestSucceed;
```
#### 4.2.2 请求播放地址失败
```
/**
*    @brief    请求播放地址失败
*/
-(void)requestFailed:(NSError *)error reason:(NSString *)reason;
```
#### 4.2.3 收到提问
```
/**
*    @brief  收到提问，用户观看时和主讲的互动问答信息
*/
- (void)onQuestionDic:(NSDictionary *)questionDic;
```
#### 4.2.4 收到回答
```
/**
*    @brief  收到回答，用户观看时和主讲的互动问答信息
*/
- (void)onAnswerDic:(NSDictionary *)answerDic;
```
#### 4.2.5 主讲和其他用户的历史互动问答信息
```
/**
*    @brief  收到提问&回答，在用户登录之前，主讲和其他用户的历史互动问答信息
*/
- (void)onQuestionArr:(NSArray *)questionArr onAnswerArr:(NSArray *)answerArr;
```
#### 4.2.6 历史聊天数据
```
/**
*    @brief  历史聊天数据
*/
- (void)onChatLog:(NSArray *)chatLogArr;
```
#### 4.2.7 主讲开始推流
```
/**
*    @brief  主讲开始推流
*/
- (void)onLiveStatusChangeStart;
```
#### 4.2.8 停止直播
```
/**
*    @brief  停止直播，endNormal表示是否异常停止推流，这个参数对观看端影响不大
*/
- (void)onLiveStatusChangeEnd:(BOOL)endNormal;
```
#### 4.2.9 收到公聊消息
```
/**
*    @brief  收到公聊消息
*/
- (void)onPublicChatMessage:(NSDictionary *)message;
```
#### 4.2.10 收到私聊信息
```
/**
*    @brief    收到私聊信息
*/
- (void)OnPrivateChat:(NSDictionary *)dic;
```
#### 4.2.11 自己被禁言
```
/*
*  @brief  收到自己的禁言消息，如果你被禁言了，你发出的消息只有你自己能看到，其他人看不到
*/
- (void)onSilenceUserChatMessage:(NSDictionary *)message;
```
#### 4.2.12 收到在线人数
```
/**
*    @brief    收到在线人数
*/
- (void)onUserCount:(NSString *)count;
```
#### 4.2.13 主讲全体禁言时，你再发消息
```
/**
*    @brief    当主讲全体禁言时，你再发消息，会出发此代理方法，information是禁言提示信息
*/
- (void)information:(NSString *)information;
```
#### 4.2.14 服务器端给自己设置的UserId
```
/**
*    @brief    服务器端给自己设置的UserId
*/
-(void)setMyViewerId:(NSString *)viewerId;
```
#### 4.2.15 收到踢出消息
```
/**
*    @brief  收到踢出消息，停止推流并退出播放（被主播踢出）
*/
- (void)onKickOut;
```
#### 4.2.16 获取房间信息
```
/**
*    @brief  获取房间信息，主要是要获取直播间模版来类型，根据直播间模版类型来确定界面布局
*    房间简介：dic[@"desc"];
*    房间名称：dic[@"name"];
*    房间模版类型：[dic[@"templateType"] integerValue];
*    模版类型为1: 聊天互动： 无 直播文档： 无 直播问答： 无
*    模版类型为2: 聊天互动： 有 直播文档： 无 直播问答： 有
*    模版类型为3: 聊天互动： 有 直播文档： 无 直播问答： 无
*    模版类型为4: 聊天互动： 有 直播文档： 有 直播问答： 无
*    模版类型为5: 聊天互动： 有 直播文档： 有 直播问答： 有
*    模版类型为6: 聊天互动： 无 直播文档： 无 直播问答： 有
*/
-(void)roomInfo:(NSDictionary *)dic;
```
#### 4.2.17 直播状态
```
/**
*    @brief  收到播放直播状态 0直播 1未直播
*/
- (void)getPlayStatue:(NSInteger)status;
```
#### 4.2.18 白板或者文档本身的宽高
```
/**
*    @brief  获取文档内白板或者文档本身的宽高，来进行屏幕适配用的
*/
- (void)getDocAspectRatioOfWidth:(CGFloat)width height:(CGFloat)height;
```
#### 4.2.19 登录成功
```
/**
*    @brief  登录成功
*/
- (void)loginSucceedPlay;
```
#### 4.2.20 登录失败
```
/**
*    @brief  登录失败
*/
-(void)loginFailed:(NSError *)error reason:(NSString *)reason;
```
#### 4.2.21 连麦相关代理方法
```
/*（连麦相关代理方法）
*  @brief WebRTC连接成功，在此代理方法中主要做一些界面的更改
*/
- (void)connectWebRTCSuccess;
```
#### 4.2.22 当前是否可以连麦
```
/*（连麦相关代理方法）
*  @brief 当前是否可以连麦
*/
- (void)whetherOrNotConnectWebRTCNow:(BOOL)connect;
```
#### 4.2.23 主播端接受连麦请求
```
/*（连麦相关代理方法）
*  @brief 主播端接受连麦请求，在此代理方法中，要调用DequestData对象的
*  - (void)saveUserInfo:(NSDictionary *)dict remoteView:(UIView *)remoteView;方法
*  把收到的字典参数和远程连麦页面的view传进来，这个view需要自己设置并发给SDK，SDK将要在这个view上进行渲染
*/
- (void)acceptSpeak:(NSDictionary *)dict;
```
#### 4.2.24  主播端发送断开连麦
```
/*（连麦相关代理方法）
*  @brief 主播端发送断开连麦的消息，收到此消息后做断开连麦操作
*/
-(void)speak_disconnect:(BOOL)isAllow;
```
#### 4.2.25 允许连麦的房间
```
/*（连麦相关代理方法）
*  @brief 本房间为允许连麦的房间，会回调此方法，在此方法中主要设置UI的逻辑，
*  在断开推流,登录进入直播间和改变房间是否允许连麦状态的时候，都会回调此方法
*/
- (void)allowSpeakInteraction:(BOOL)isAllow;
```
#### 4.2.26 切换源
```
/*
*  @brief 切换源，firRoadNum表示一共有几个源，secRoadKeyArray表示每
*  个源的描述数组，具体参见demo，firRoadNum是下拉列表有面的tableviewcell
*  的行数，secRoadKeyArray是左面的tableviewcell的描述信息数组
*/
- (void)firRoad:(NSInteger)firRoadNum secRoadKeyArray:(NSArray *)secRoadKeyArray;
```
#### 4.2.27 自定义消息
```
/*
*  自定义消息
*/
- (void)customMessage:(NSString *)message;
```
#### 4.2.28 公告
```
/*
*  公告
*/
- (void)announcement:(NSString *)str;
```
#### 4.2.29 监听到有公告消息
```
/*
*  监听到有公告消息
*/
- (void)on_announcement:(NSDictionary *)dict;
```
#### 4.2.30 开始抽奖
```
/*
*  开始抽奖
*/
- (void)start_lottery;
```
#### 4.2.31 抽奖结果
```
/*
*  抽奖结果
*/
- (void)lottery_resultWithCode:(NSString *)code myself:(BOOL)myself winnerName:(NSString *)winnerName remainNum:(NSInteger)remainNum;
```
#### 4.2.32 退出抽奖
```
/*
*  退出抽奖
*/
- (void)stop_lottery;
```
#### 4.2.33 开始签到
```
/*
*  开始签到
*/
- (void)start_rollcall:(NSInteger)duration;
```
#### 4.2.34 开始答题
```
/*
*  开始答题
*/
- (void)start_vote:(NSInteger)count singleSelection:(BOOL)single;
```
#### 4.2.35 结束答题
```
/*
*  结束答题
*/
- (void)stop_vote;
```
#### 4.2.36 答题结果
```
/*
*  答题结果
*/
- (void)vote_result:(NSDictionary *)resultDic;
```
#### 4.2.37  加载视频失败
```
/*
*  加载视频失败
*/
- (void)play_loadVideoFail;
```
#### 4.2.38 接收到发送的广播
```
/*
*  接收到发送的广播
*/
- (void)broadcast_msg:(NSDictionary *)dic;
```
#### 4.2.39 发布问题的ID
```
/*
*  发布问题的ID
*/
- (void)publish_question:(NSString *)publishId;
```
#### 4.2.40 发布问卷
```
/*
*  发布问卷
*/
- (void)questionnaire_publish;
```
#### 4.2.41 结束发布问卷
```
/*
*  结束发布问卷
*/
- (void)questionnaire_publish_stop;
```
#### 4.2.42 问卷详细内容
```
/*
*  获取问卷详细内容
*/
- (void)questionnaireDetailInformation:(NSDictionary *)detailDic;
```
#### 4.2.43 提交问卷结果
```
/*
*  提交问卷结果（成功，失败）
*/
- (void)commitQuestionnaireResult:(BOOL)success;
```
### 4.3 事件监听
**添加播放器监听事件（监听播放状态改变）**
```
1. 播放器的监听事件有以下几种
1. IJKMPMoviePlayerLoadStateDidChangeNotification        视频加载视频状态发生改变
（1）IJKMPMovieLoadStateStalled    正在加载
（2）IJKMPMovieLoadStatePlayable  可以播放
（3）IJKMPMovieLoadStatePlaythroughOK 已经缓存完成,如果设置了自动播放,这时会自动播放
2. IJKMPMoviePlayerPlaybackStateDidChangeNotification   视频播放状态发生改变
（1）IJKMPMoviePlaybackStateStopped 播放停止
（2）IJKMPMoviePlaybackStatePlaying 播放
（3）IJKMPMoviePlaybackStatePaused  暂停
（4）IJKMPMoviePlaybackStateInterrupted 打断（停止，中断）
（5）IJKMPMoviePlaybackStateSeekingForward 向前拖动
（6）IJKMPMoviePlaybackStateSeekingBackward 向后拖动
3. IJKMPMoviePlayerPlaybackDidFinishReasonUserInfoKey   视频播放终止播放原因
4. IJKMPMoviePlayerPlaybackDidFinishNotification         视频终止播放  
5. IJKMPMovieNaturalSizeAvailableNotification    视频开始播放时，用这个可以监听到视频的分辨率
6. IJKMPMoviePlayerPlaybackDidFinishReasonUserInfoKey  视频播放完成(有可能是加载失败，也有可能是播放结束)
7. IJKMPMoviePlayerPlaybackDidFinishNotification  视频播放完成(有可能是加载失败，也有可能是播放结束)
```
#### 4.4 接收数据格式介绍

1.  收到提问&回答(RequestDataDelegate和RequestDataPlayBackDelegate 中代理方法)
```
RequestDataDelegate中代理名称如下：
- (void)onQuestionArr:(NSArray *)questionArr onAnswerArr:(NSArray *)answerArr;

RequestDataPlayBackDelegate中代理名称如下：
- (void)onParserQuestionArr:(NSArray *)questionArr onParserAnswerArr:(NSArray *)answerArr;
```
2.  提问格式，数组
```
(
{
content = question;//问题内容
encryptId = 2264AF619DD010E9;//问题ID
id = 342854;//提问ID，此ID和上面的encryptId无差别，都是提问的唯一标志，且和下面的回答id是一对多对应，一个问题可以有多条回答信息
isPublish = 1;//问题是否发布
questionUserAvatar = "";//头像
questionUserId = 00d30351184d4cbdb0fcbadd9a55ba1c;//提问问题的用户的用户id
questionUserName = qwer;//提问问题用户的用户名
time = 69006;//提问问题时间点（毫秒）
triggerTime = "2017-11-21 13:50:19";//具体时间点
}
)
```
3. 收到提问(RequestDataDelegate中代理方法)
```
- (void)onQuestionDic:(NSDictionary *)questionDic;
{
action = question;
time = 0;//新的提问和回答时间全部为0，历史的提问和回答才有正确时间
value =     {
content = question;
id = BC2D248818827729;//如果该用户被禁言后，该用户发出问题的id都为-1，并且该问题只会发送给该用户，不会发送给其他用户
triggerTime = "2017-11-21 14:02:54";
userAvatar = "";
userId = 9c408a9761d74cb9b64587b2776c5327;
userName = qwer;
};
}
注：提问者角色默认为学生
```
4 收到回答(RequestDataDelegate中代理方法)
```
- (void)onAnswerDic:(NSDictionary *)answerDic;
{
action = answer;
time = 0;//新的提问和回答时间全部为0，历史的提问和回答才有正确时间
value =     {
content = answer;
isPrivate = 0;
questionId = BC2D248818827729;
questionUserId = 9c408a9761d74cb9b64587b2776c5327;
triggerTime = "2017-11-21 14:04:17";
userAvatar = "";
userId = 4c70191006d54d549e43ccff4bc1bfd1;
userName = admin;
userRole = publisher;
};
}
```
5 收到30条历史聊天数据信息(RequestDataDelegate中代理方法)
```
- (void)onChatLog:(NSArray *)chatLogArr;
(
{
content = message;//聊天内容
time = 68996;//聊天时间（毫秒）
userAvatar = "";//聊天者头像
userCustomMark = "";//个性化信息
userId = 00d30351184d4cbdb0fcbadd9a55ba1c;//聊天者id
userName = qwer;//聊天者用户名
userRole = student;//聊天者用户角色
}
)
```
6 收到学生发送的公聊信息(RequestDataDelegate中代理方法)
```
- (void)onPublicChatMessage:(NSDictionary *)dic;
{
msg = chat;//内容
time = "14:10:16";//时间
useravatar = "";//头像
usercustommark = "";//个性化信息
userid = 9c408a9761d74cb9b64587b2776c5327;//用户id
username = qwer;//用户名
userrole = student;//用户角色
}
```
7 收到学生禁言消息(RequestDataDelegate中代理方法)

- (void)onSilenceUserChatMessage:(NSDictionary *)message;
{
msg = asdfghj;
time = "14:13:53";
useravatar = "";
usercustommark = "";
userid = 9c408a9761d74cb9b64587b2776c5327;
username = qwer;
userrole = student;
}

8 收到学生私聊(RequestDataDelegate中代理方法)

- (void)OnPrivateChat:(NSDictionary *)dic;
{
fromuseravatar = "";
fromuserid = 9c408a9761d74cb9b64587b2776c5327;
fromusername = qwer;
fromuserrole = student;
usercustommark = "";
msg = chat;
time = "14:11:32";
touserid = 00d30351184d4cbdb0fcbadd9a55ba1c;
}

9 聊天信息数组(RequestDataPlayBackDelegate中代理方法)

-(void)onParserChat:(NSArray *)chatArr;

(
{
content = aaa;//聊天内容
time = 27;//聊天的时间点
userAvatar = "";//聊天用户头像
userCustomMark = "";//个性化信息
userId = 00942719153943a4a847d8c9278f030d;//聊天用户id
userName = admin;//聊天用户名
userRole = publisher;//聊天用户角色
}
)


## 5.API查询

Doc目录打开index.html文件

## 6.Q&A

### 6.1如发现 dyld: Library not loaded: @rpath/CCPush.framework/CCPush
Referenced from: /var/containers/Bundle/Application/3CC41924-7F22-48FC-AAD1-7C2C089EF749/demo.app/demo
Reason: image not found问题
将报错的类库添加进入
General -> Embedded Binaries 和 General -> Linked Frameworks and Libraries 两处即可

