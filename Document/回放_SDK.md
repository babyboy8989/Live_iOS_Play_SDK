[TOC]
## 1.概述
iLive_iOS_Play_SDK 是一个适用于iOS平台的直播SDK，使用此SDK可以与CC视频云服务进行对接，在iOS端使用CC视频的直播回放功能,本SDK只针对iOS手机播放端。

### 1.1 功能特性
| 功能         | 描述                 |
| :----------- | :------------------- |
| 回放视频     | 观看直播回放视频     |
| 文档展示     | 观看直播时展示的文档 |
| 问答         | 观看直播时的问答     |
| 历史聊天数据 | 观看历史聊天数据     |
| 房间信息     | 观看直播间信息       |

### 1.2 阅读对象
本文档为技术文档，需要阅读者：
* 具备基本的iOS开发能力
* 准备接入CC视频的直播回放SDK相关功能
* 对CC云直播产品使用方法有基础的了解，[使用帮助地址](http://doc.bokecc.com/live/manual/introduction/)


## 2.开发准备

### 2.1 开发环境
* Xcode  : Xcode  开发IDE
### 2.2 错误码及错误描述
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


## 3.快速集成

首先，需要下载最新版本的SDK，下载地址为：[Live_iOS_Play_SDK](https://github.com/CCVideo/Live_iOS_Play_SDK)
集成前需要知道的事情:
1. 工程分三个部分：一个有连麦功能（Demo-观看端有连麦），体积较大，一个无连麦功能（Demo-观看端无连麦），体积较小，第三个部分是离线下载回放（Demo-离线下载），其中离线下载回放中的CCSDK用有连麦的库和无连麦的库都可以，示例中用的无连麦的库(体积较小)
2. （重要）本SDK只支持真机，不支持模拟器
3. （重要）本SDK只支持iOS8.0及以上版本，iOS7.x版本不支持
4. （重要）当使用有连麦工程时，分为音视频连麦和音频连麦两种连麦，有连麦的demo中没有以页面体现两种连麦的按钮，当选中简介时，连麦为音视频连麦，当选中 文档，聊天或问答任意一个时，连麦为音频连麦，代码使中用_isAudioVideo加以区分，当_isAudioVideo==YES;时为音视频连麦，当_isAudioVideo==NO;时为音频连麦
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



首先导入头文件
```
#import "CCSDK/CCLiveUtil.h"
#import "CCSDK/RequestDataPlayBack.h"
```
配置参数
```
//更多参数请查看PlayParameter 属性
PlayParameter *parameter = [[PlayParameter alloc] init];
parameter.userId = @"用户ID";
parameter.roomId = @"房间ID";
parameter.liveId = @"直播ID，回放时才用到";
parameter.recordId = @"回放ID";
parameter.viewerName = @"用户名称";
parameter.token = @"房间密码";
parameter.security = @"是否使用https，静态库暂时只能使用http协议";
parameter.pauseInBackGround = YES;后台是否继续播放
_requestDataPlayBack = [[RequestDataPlayBack alloc] initWithParameter:parameter];
_requestDataPlayBack.delegate = self;;//需要实现RequestDataPlayBackDelegate代理方法
```
实现代理方法
```
#pragma mark - RequestDataPlayBackDelegate
/**
*    @brief    请求成功
*/
-(void)requestSucceed {
//    NSLog(@"请求成功！");
}

/**
*    @brief    登录请求失败
*/
-(void)requestFailed:(NSError *)error reason:(NSString *)reason {
NSString *message = nil;
if (reason == nil) {
message = [error localizedDescription];
} else {
message = reason;
}
}
```
### 3.5 开启视频直播回放
跳转到直播回放页面,配置观看直播回放同样需要导入头文件
```
#import "CCSDK/CCLiveUtil.h"
#import "CCSDK/RequestDataPlayBack.h"
```
配置参数,和登录时配置的参数差不多,很多参数都是登录传过来的
```
PlayParameter *parameter = [[PlayParameter alloc] init];
parameter.userId = @"用户ID";
parameter.roomId = @"房间ID";
parameter.liveId = @"直播ID，回放时才用到";
parameter.recordId = @"回放ID";
parameter.viewerName = @"用户名称";
parameter.token = @"房间密码";
parameter.playerParent = self.videoView;//视频父类窗口
parameter.playerFrame = _videoRect//视频区域;
parameter.security = NO;
parameter.pauseInBackGround = YES;
parameter.defaultColor = [UIColor whiteColor];
parameter.scalingMode = 1;
_requestDataPlayBack = [[RequestDataPlayBack alloc] initWithParameter:parameter];
_requestDataPlayBack.delegate = self;
```

### 3.6 文档观看
PlayParameter增加两个属性
```
@property(nonatomic,strong)UIView                       *docParent;//文档父类窗口
@property(nonatomic,assign)CGRect                       docFrame;//文档区域
```
## 4. 功能使用
### 4.1 RequestDataPlayBack回放请求类介绍

#### 4.1.1 在回放PlayBackViewController.m中主要调用此类来和此SDK交互

#### 4.1.2 RequestDataPlayBack中的成员变量

@property (weak,nonatomic) id<RequestDataPlayBackDelegate>  delegate;//代理
@property (retain,    atomic) id<IJKMediaPlayback>          ijkPlayer;//播放器

播放器倍速，直播和回放都可以用，一般只用于回放

ijkPlayer.playbackRate = 1.5;             设置播放器以1.5倍速播放，其他倍速同理，可快可慢
float speed = ijkPlayer.playbackRate;     获取当前播放器的播放速度，并赋值给speed变量

#### 4.1.3 RequestDataPlayBack中的成员方法

/**
*    @brief    登录房间
*    @param     parameter   配置参数信息
*  必填参数 userId
*  必填参数 roomId
*  必填参数 liveid
*  必填参数 viewerName
*  必填参数 token
*  必填参数 security
*/
- (id)initLoginWithParameter:(PlayParameter *)parameter;

/**
*    @brief    进入房间，并请求画图聊天数据并播放视频（可以不登陆，直接从此接口进入直播间）
*    @param     parameter   配置参数信息
*  必填参数 userId;
*  必填参数 roomId;
*  必填参数 liveid;
*  必填参数 viewerName;
*  必填参数 token;
*  必填参数 docParent;
*  必填参数 docFrame;
*  必填参数 playerParent;
*  必填参数 playerFrame;
*  必填参数 security;
*  必填参数 pauseInBackGround;
*  必填参数 defaultColor;
*  必填参数 PPTScalingMode;
*  必填参数 scalingMode;
*/
- (id)initWithParameter:(PlayParameter *)parameter;

/**
*    @brief    销毁文档和视频，清除视频和文档的时候需要调用,推出播放页面的时候也需要调用
*/
- (void)requestCancel;

/**
*    @brief    time：从直播开始到现在的秒数，SDK会在画板上绘画出来相应的图形
*/
- (void)continueFromTheTime:(NSInteger)time;

/**
*    @brief  获取文档区域内白板或者文档本身的宽高比，返回值即为宽高比，做屏幕适配用
*/
- (CGFloat)getDocAspectRatio;

/**
*    @brief  改变文档区域大小,主要用在文档生成后改变文档窗口的frame
*/
- (void)changeDocFrame:(CGRect) docFrame;

/**
*    @brief  改变播放器frame
*/
- (void)changePlayerFrame:(CGRect) playerFrame;

/**
*    @brief  播放器暂停  
*/
- (void)pausePlayer;

/**
*    @brief  播放器播放
*/
- (void)startPlayer;

/**
*    @brief  播放器关闭
*/
- (void)shutdownPlayer;

/**
*    @brief  播放器停止
*/
- (void)stopPlayer;

/**
*    @brief  从头播放
*/
- (void)replayPlayer;

/**
*    @brief  播放器是否播放
*/
- (BOOL)isPlaying;

/*
*  @brief  播放器当前播放时间
*/
- (NSTimeInterval)currentPlaybackTime;

/*
*  @brief  设置播放器当前播放时间（用于拖拽进度条时掉用的）
*/
- (void)setCurrentPlaybackTime:(NSTimeInterval)time;

/*
*  @brief 回放视频总时长
*/
- (NSTimeInterval)playerDuration;

/**
*    @brief  设置后台是否可播放
*/
- (void)setpauseInBackGround:(BOOL)pauseInBackGround;

### 4.2 事件监听
事件监听只需要实现对应的代理方法即可
#### 4.2.1  获取文档内白板或者文档本身的宽高
```
/**
*    @brief  获取文档内白板或者文档本身的宽高，来进行屏幕适配用的
*/
- (void)getDocAspectRatioOfWidth:(CGFloat)width height:(CGFloat)height;
```
#### 4.2.2 历史提问&回答
```
/**
*    @brief    收到本房间的历史提问&回答
*/
- (void)onParserQuestionArr:(NSArray *)questionArr onParserAnswerArr:(NSArray *)answerArr;
```
####  4.2.3 历史聊天数据
```
/**
*    @brief    解析本房间的历史聊天数据
*/
-(void)onParserChat:(NSArray *)arr;
```
#### 4.2.4 请求回放地址成功
```
/**
*    @brief    请求回放地址成功
*/
-(void)requestSucceed;
```
#### 4.2.5 请求回放地址失败
```
/**
*    @brief    请求回放地址失败
*/
-(void)requestFailed:(NSError *)error reason:(NSString *)reason;
```
#### 4.2.6 登录成功
```
/**
*    @brief  登录成功
*/
- (void)loginSucceedPlayBack;
```
#### 4.2.7 登录失败
```
/**
*    @brief  登录失败
*/
-(void)loginFailed:(NSError *)error reason:(NSString *)reason;
```
#### 4.2.8  获取房间信息
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
#### 4.2.9 加载视频失败
```
/*
*  加载视频失败
*/
- (void)playback_loadVideoFail;
```
#### 4.2.10 回放翻页数据列表
```
/*
*  回放翻页数据列表
*  docName        文档名称
*  url         文档url
*  pageTitle    页面标题
*  time        翻页时间
*/
- (void)pageChangeList:(NSMutableArray *)array;
```
### 4.3.事件监听
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
### 4.4 接收数据格式介绍

1  收到提问&回答(RequestDataDelegate和RequestDataPlayBackDelegate 中代理方法)
```
RequestDataDelegate中代理名称如下：
- (void)onQuestionArr:(NSArray *)questionArr onAnswerArr:(NSArray *)answerArr;

RequestDataPlayBackDelegate中代理名称如下：
- (void)onParserQuestionArr:(NSArray *)questionArr onParserAnswerArr:(NSArray *)answerArr;
```
2  提问格式，数组
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
3 收到提问(RequestDataDelegate中代理方法)
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
