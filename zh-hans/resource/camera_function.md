# 摄像头功能

### 操作失败回调

摄像头相关操作都是异步，操作的结果通过 ``` TuyaSmartCameraDelegate ``` 的代理方法返回。下面所述中的 “成功回调” 皆是指操作成功时调用的代理方法。当操作失败时，会统一由以下方法回调。

```objective-c
- (void)camera:(id<TuyaSmartCameraType>)camera didOccurredError:(TYCameraErrorCode)errCode;
```

errCode 表示哪个操作失败。具体errCode的定义如下：

```objective-c
typedef enum {
TY_ERROR_NONE,                      // 0
TY_ERROR_CONNECT_FAILED,            // 1    连接失败（包含超时，密码错误，不支持的设备等）
TY_ERROR_CONNECT_DISCONNECT,        // 2    连接断开
TY_ERROR_ENTER_PLAYBACK_FAILED,     // 3    连接回放通道失败
TY_ERROR_START_PREVIEW_FAILED,      // 4    预览失败（如果在预览过程中，连接断开，也会回调这个错误）
TY_ERROR_START_PLAYBACK_FAILED,     // 5    回放失败（如果在回放过程中，回放通道的连接断开，也会回调这个错误）
TY_ERROR_PAUSE_PLAYBACK_FAILED,     // 6    暂停回放失败
TY_ERROR_RESUME_PLAYBACK_FAILED,    // 7    恢复回放失败
TY_ERROR_ENABLE_MUTE_FAILED,        // 8    设置静音状态失败（通常在关闭静音，也就是开启声音监听的时候才会发生错误）
TY_ERROR_START_TALK_FAILED,         // 9    开启对讲失败
TY_ERROR_SNAPSHOOT_FAILED,          // 10   截图失败
TY_ERROR_RECORD_FAILED,             // 11   录制失败（开启录制和停止录制的操作都可能会触发这个错误）
TY_ERROR_ENABLE_HD_FAILED,          // 12   设置高清标清状态失败
TY_ERROR_GET_HD_FAILED,             // 13   获取高清标清状态失败
TY_ERROR_QUERY_RECORD_DAY_FAILED,   // 14   查询有视频记录的日期失败
TY_ERROR_QUERY_TIMESLICE_FAILED,    // 15   查询某天的所有视频记录失败
} TYCameraErrorCode;
```


### 视频播放视图

* 获取视频播放视图

```objective-c
- (void)viewDidLoad {
CGFloat ScreenWidth = [UIScreen mainScreen].bounds.size.width;
CGFloat ScreenHeight = [UIScreen mainScreen].bounds.size.height;
CGFloat VideoWidth = ScreenWidth;
CGFloat VideoHeight = VideoWidth * 9 / 16;

UIView<TuyaSmartVideoViewType> *videoView = [self.camera videoView];
videoView.frame = CGRectMake(0, 64, VideoWidth, VideoHeight);
[self.view addSubview:videoView];
}
```

* 视频缩放、移动、清除操作

```objective-c
// 将视频图像放大1.2倍
[videoView tuya_setScaled:1.2];
// 将视频图像向右移动10.0个pt点
[videoView tuya_setOffset:CGPointMake(10.0, 0.0)];
// 清除图像，并将缩放倍数，和图像位移重置
[videoView tuya_clear];
```
### 连接p2p通道

```objective-c
// 连接命令
- (void)viewDidLoad {
	// other code...
	[self.camera connect];
}

// 成功回调
- (void)cameraDidConnected:(id<TuyaSmartCameraType>)camera {
	self.isConnected = YES;
}

// 连接断开回调
- (void)cameraDisconnected:(id<TuyaSmartCameraType>)camera {
	self.isConnected = NO;
	NSLog(@"---disconnected");
}
```
### 连接回放通道

p2p连接需要建立两个通道，默认通道用于命令下发与预览的音视频数据传输，若要回放功能，需要在默认通道连接后，再建立一个回放通道，用于回放的音视频传输。

```objective-c
- (void)cameraDidConnected:(id<TuyaSmartCameraType>)camera {
	self.isConnected = YES;
	// 建立回放通道
	[self.camera enterPlayback];
}

// 成功回调
- (void)cameraDidConnectPlaybackChannel:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---didconnectedplayback");
}
```
### 播放模式

定义预览还是回放，在设置静音状态时，需要传入此参数，以指定设置哪个模式是否静音。

```objective-c
typedef NS_ENUM(NSUInteger, TuyaSmartCameraPlayMode) {
	TuyaSmartCameraPlayModeNone,        // 无
	TuyaSmartCameraPlayModePreview,     // 预览
	TuyaSmartCameraPlayModePlayback     // 回放
};
```
### 开启预览

```objective-c
- (void)startPreview {
	// 判断是否在录制视频，如果正在录制视频，停止录制。(是否在录制中的状态由开发者自己维护)
	if (self.isRecording) {
		[self.camera stopRecord];
	}
	// 判断是否正在预览，如果正在预览，停止预览。(是否在预览中的状态由开发者自己维护)
	if (self.isPlaybacking) {
		[self.camera stopPlayback];
	}
	// 判断p2p通道是否已经连接，如果未连接或者连接已经断开，连接通道。(p2p通道是否连接的状态由开发者自己维护)
	if (!self.isConnected) {
		[self.camera connect];
	}
		// 开启预览
		[self.camera startPreview];
	}
// 成功回调
- (void)cameraDidBeginPreview:(id<TuyaSmartCameraType>)camera {
	// 当前的播放模式
	self.playMode = TuyaSmartCameraPlayModePreview;
	// 预览状态
	self.isPreviewing = YES;
}
```
* 注：当前摄像头的状态，如正在预览，录制，回放，p2p通道已连接，连接断开等，由开发者自己维护，SDK中不会保留这些状态值，只负责命令的下发与回调。下面不再赘述。


### 停止预览

```objective-c
- (void)stopPreview {
	// 若正在录制视频，停止录制
	if (self.isRecording) {
		[self.camera stopRecord];
	}
	// 如果正在对讲，则关闭对讲
	if (self.isTalking) {
		[self.camera startTalk];
	}
	// 停止预览，此操作不会失败
	[self.camera stopPreview];
	self.isPreviewing = NO;
	self.playMode = TuyaSmartCameraPlayModeNone;
}

// 成功回调
- (void)cameraDidStopPreview:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---stop preview");
}
```
### 获取有回放视频记录的日期

在开始回放前，需要获取到回放视频记录的信息。首先获取有回放视频记录的日期

```objective-c
- (void)queryPlaybackDays {
	// 根据年份和月份查询某一月中有回放视频记录的日期
	[self.camera queryRecordDaysWithYear:2018 month:7];
}

// 成功回调
- (void)camera:(id<TuyaSmartCameraType>)camera didReceiveRecordDayQueryData:(NSArray<NSNumber *> *)days {
// days是一个 NSNumber 类型的日期数组，包含有回放视频记录的日期，日期的值从 1 到 31 之间
	self.playbackDays = days;
	self.curentDay = days.lastObject.integerValue;
}
```
### 获取某日的视频回放信息

获取到有用回放记录的日期后，根据日期获取当日的视频回放记录

```objective-c
- (void)queryPlaybackRecords {
	// 根据日期查询回放视频信息
	[self.camera queryRecordTimeSliceWithYear:2018 month:7 day:self.curentDay];
}

// 成功回调
- (void)camera:(id<TuyaSmartCameraType>)camera didReceiveTimeSliceQueryData:(NSArray<NSDictionary *> *)timeSlices {
// timeSlices 是一个字典数组，包含当日所有的回放视频片段信息。
// 每个字典对应一个回放视频片段，字典包含四个字段：
/*
/// 回放片段开始时间的key, 值类型是 NSDate，基本手机系统本地时间的时区
FOUNDATION_EXTERN NSString * const kTuyaSmartTimeSliceStartDate;

/// 回放片段结束时间的key, 值类型是 NSDate，基本手机系统本地时间的时区
FOUNDATION_EXTERN NSString * const kTuyaSmartTimeSliceStopDate;

/// 回放片段开始时间的key, 值类型是 NSNumber，Unix时间戳，基于格林时间
FOUNDATION_EXTERN NSString * const kTuyaSmartTimeSliceStartTime;

/// 回放片段结束时间的key, 值类型是 NSNumber，Unix时间戳，基于格林时间
FOUNDATION_EXTERN NSString * const kTuyaSmartTimeSliceStopTime;
*/
	self.timeSlices = timeSlices;
}
```
### 开启回放

```objective-c
- (void)startPlayBack:(NSDictionary)timeSlice {
	// 如果正在预览，则停止预览。包括停止录制，对讲等操作。详见上述的停止预览方法。
	if (self.isPreviewing) {
		[self stopPreview];
	}
	// timeSlice 取自上面视频片段回放信息数组里的一个元素
	NSInteger startTime = [[timeSlice objectForKey:kTuyaSmartTimeSliceStartTime] integerValue];
	NSInteger stopTime = [[timeSlice objectForKey:kTuyaSmartTimeSliceStopTime] integerValue];
	// 开始回放视频片段，传入开始播放的时间戳，视频片段的开始时间戳，结束时间戳
	// 开始播放的时间戳需要在视频片段的开始时间与结束时间之间，这里默认从开始时间开始播放
	[self.camera startPlayback:startTime startTime:startTime stopTime:stopTime];
}

// 成功回调
- (void)cameraDidBeginPlayback:(id<TuyaSmartCameraType>)camera {
	self.playMode = TuyaSmartCameraPlayModePlayback;
	self.isPlaybacking = YES;
}
```
### 暂停回放

```objective-c
- (void)pausePlayback {
	// 如果正在录制，停止录制
	if (self.isRecording) {
		[self.camera stopRecord];
	}
	// 暂停回放
	[self.camera pausePlayback];
}

// 成功回调
- (void)cameraDidPausePlayback:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---pause playback");
}
```
### 恢复回放

```objective-c
- (void)resumePlayback {
	// 恢复回放
	[self.camera resumePlayback];
}

// 成功回调
- (void)cameraDidResumePlayback:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---resume playback");
}
```
### 停止回放

```objective-c
- (void)stopPlayback {
	// 如果正在录制，停止录制
	if (self.isRecording) {
		[self.camera stopRecord];
}
	// 停止回放
	[self.camera stopPlayback];
}

// 成功回调
- (void)cameraDidStopPlayback:(id<TuyaSmartCameraType>)camera {
	self.playMode = TuyaSmartCameraPlayModeNone;
	self.isPlaybacking = NO;
}
```
### 回放结束回调

```objective-c
// 视频片段回放结束时，会触发此回调。有些设备会自动播放当天的下一个视频片段，直到当天的所有视频片段播放完成后，才会触发这个回调。什么时候触发，取决设备固件的实现。
- (void)cameraPlaybackDidFinished:(id<TuyaSmartCameraType>)camera {
	self.playMode = TuyaSmartCameraPlayModeNone;
	self.isPlaybacking = NO;
}
```
### 接收到第一帧视频回调

```objective-c
// 预览或者回放开始后，首次接收到视频帧的时候，会触发这个回调。此时表示视频开始正常播放了。
- (void)cameraDidReceiveFirstFrame:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---receive first frame");
}
```
### 开启视频录制

```objective-c
- (void)startRecord {
	// 如果没有播放任何视频，或者已经在录制中，不做任何操作
	if (self.playMode == TuyaSmartCameraPlayModeNone || self.isRecording) {
		return;
	}

	// 如果正在对讲，则关闭对讲
	if (self.isTalking) {
		[self.camera startTalk];
	}

	// 开启录制
	[self.camera startRecord];
}

// 成功回调
- (void)cameraDidStartRecord:(id<TuyaSmartCameraType>)camera {
	self.isRecording = YES;
}
```
### 停止视频录制

```objective-c
- (void)stopRecord {
	// 如果正在录制视频，停止录制
	if (self.isRecording) {
		[self.camera stopRecord];
	}
}

// 成功回调
- (void)cameraDidStopRecord:(id<TuyaSmartCameraType>)camera {
	self.isRecording = NO;
}
```
* 注：如果视频录制成功，会直接保存在手机相册。同时，会在手机相册中创建一个与APP同名的自定义相册，视频将会出现在这个自定义相册中。


### 视频截图

```objective-c
- (void)snapShoot {
	// 如果没有播放任何视频，不做任何操作
	if (self.playMode == TuyaSmartCameraPlayModeNone) {
		return;
	}
	// 截图操作
	[self.camera snapShoot];
}

// 成功回调
- (void)camearaSnapShootSuccess:(id<TuyaSmartCameraType>)camera {
	NSLog(@"---snap shoot success");
}
```
* 注：如果截图成功，图片会直接保存在手机相册。同时，会在手机相册中创建一个与APP同名的自定义相册，截图将会出现在这个自定义相册中。


### 设置静音

```objective-c
- (void)enabelMute:(BOOL)isMuted {
	// 如果当前没有播放视频，则不做任何操作
	if (self.playMode == TuyaSmartCameraPlayModeNone) {
		return;
	}
	// 给当前的播放模式设置静音状态
	// isMuted: YES-关闭视频声音；NO-开启视频声音
	// 视频的声音默认是关闭的
	[self.camera enableMute:isMuted forPlayMode:self.playMode];
}

// 静音状态改变回调
- (void)camera:(id<TuyaSmartCameraType>)camera didReceiveMuteState:(BOOL)isMute playMode:(TuyaSmartCameraPlayMode)playMode {
	self.isMuted = isMute;
}
```
### 清晰度获取与更改

```objective-c
// 预览开启成功的回调
- (void)cameraDidBeginPreview:(id<TuyaSmartCameraType>)camera {
	// other code ...
	// 发送查询清晰度的命令
	[self.camera getHD];

	// 更改清晰度
	// [self.camera enableHD:YES];
}

// 清晰度状态回调，查询或设置清晰度的结果，统一由这个代理方法回调
- (void)camera:(id<TuyaSmartCameraType>)camera didReceiveDefinitionStatus:(BOOL)isHd {
	// isHd: 是否高清；YES-高清；NO-标清
	self.isHd = isHd;
}
```
### 开启对讲

```objective-c
- (void)startTalk {
	// 如果正在录制，停止录制
	if (self.isRecording) {
		[self.camera stopRecord];
	}

	// 发送开启对讲的命令
	[self.camera startTalk];
}

// 成功回调
- (void)cameraDidBeginTalk:(id<TuyaSmartCameraType>)camera {
	self.isTalking = YES;
}
```
### 停止对讲

```objective-c
- (void)stopTalk {
	// 如果正在对讲，发送停止对讲的命令
	if (self.isTalking) {
		[self.camera stopTalk];
	}
}

// 成功回调
- (void)cameraDidStopTalk:(id<TuyaSmartCameraType>)camera {
	self.isTalking = NO;
}
```
* 注：对讲和录制是互斥的，而且只有在预览时可以开启对讲。


### 分辨率改变回调

```objective-c
// 视频图像分辨率更改的回调
- (void)camera:(id<TuyaSmartCameraType>)camera resolutionDidChangeWidth:(NSInteger)width height:(NSInteger)height {
	NSLog(@"---resolution changed: %ld x %ld", width, height);
}
```
### 开启裸流。如果需要获取原始的视频帧数据，可以开启获取裸流属性。

```objective-c
- (void)viewDidLoad {
	// other init code
	// 当camera.isRecvFrame 为YES时，将不会自动渲染视频画面，视频帧数据将通过下面的代理方法给出。
	self.camera.isRecvFrame = YES;
}

/**
获取原始视频帧数据的代理方法。
@param camera      摄像头对象
@param frameData   原始视频帧数据
@param size        视频帧数据大小
@param frameInfo   视频帧信息，包含编码方式和时间戳
*/
- (void)camera:(id<TuyaSmartCameraType>)camera ty_didReceiveFrameData:(const char *)frameData dataSize:(unsigned int)size frameInfo:(TuyaSmartVideoFrameInfo)frameInfo;

```
* 注:  开启获取裸流后,原始的渲染视图(videoVIew)就不会渲染图像,需要开发者自行通过该接口的数据进行解析渲染。


### 销毁资源

```objective-c
// 摄像头面板销毁时，销毁摄像头资源。
- (void)dealloc {
	[self.camera destory];
}
```

