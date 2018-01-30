### 6.2.2 LFLiveKit
LFLiveKit框架是优酷土豆旗下实现直播推流的开源框架，利用H264和AAC硬编码，支持GPUImage美化，rtmp传输推流，弱网络丢帧，可以动态切换速率。

####LFLiveKit的导入

1.LFLiveKit导入之前，需要先导入一些相关类库：UIKit、Foundation、AVFoundation、VideoToolbox、AudioToolbox、libz、libstdc++。
2.导入依然有两种方式
* 直接从github上下载 https://github.com/LaiFengiOS/LFLiveKit
* 使用Cocoapods导入
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '7.0'
pod 'LFLiveKit'
```

####LFLiveKit的基础架构

| 类名 | 说明 |
| :--- | :--- |
| LFLiveSession | 是整个sdk核心，提供对外部的主要接口。其主要功能有：管理推流开关，管理音视频录制及渲染，管理录制渲染后的音视频编吗，管理编吗后的数据上传，管理音视频的基础配置，回调推流状态和异常上报等。 |
| LFLiveAudioConfiguration | 音频配置，配置相关音频信息\(音频质量，码率，采样率，声道数\) |
| LFLiveVideoConfiguration | 视频配置，配置相关音频基本信息\(视频质量，码率，帧数，分辨率\)和应用配置如最大最小帧率等。 |
| LFVideoCapture | 视频管理类，管理视频的输入和输出。同时处理业务需求如：美颜，亮度，水印等效果。用了一个第三方：GPUIImage处理渲染效果 |
| LFAudioCapture | 音频管理，管理音频的输入开关。这一块儿没有多大的定制，应用的原生的API即可。 |
| LFH264VideoEncoder,LFHardwareVideoEncoder | 视频编码类，分别对应8.0以前和8.0以后的两种设备的视频编码类。都遵守LFVideoEncoding协议，并设置LFStreamSocketDelegate协议给session管理 |
| LFHardwareAudioEncoder | 音频编码类，遵守LFVideoEncoding协议，并设置LFStreamSocketDelegate协议给session管理 |
| LFFrame | 数据信息的基类，作为上传到服务器数据的基本模型 |
| LFVideoFrame | 视频信息，作为上传到服务器视频数据的模型 |
| LFAudioFrame | 音频信息，作为上传到服务器音频数据的模型 |
| LFLiveStreamInfo | 推流信息：推流地址\(目前主要应用rtmp推流\)；流状态；音视频配置信息；异常信息 |
| LFStreamRTMPSocket | 数据上传管理类：开关数据上传，回调连接状态和异常。遵循LFStreamSocket协议，并设置LFStreamSocketDelegate给session管理 |
| LFLiveDebug | 调试信息：这个是开发时候的内部表示，主要用于记录调试作用。 |
| LFStreamingBuffer | 本地采样：通过本地采样监控缓冲区，可实现相关切换帧率码率等策略 |

####LFLiveKit的简单使用

<pre><code>
- (LFLiveSession*)session {
    if (!_session) {
        _session = [[LFLiveSession alloc] initWithAudioConfiguration:[LFLiveAudioConfiguration defaultConfiguration] videoConfiguration:[LFLiveVideoConfiguration defaultConfiguration]];
        _session.preView = self;
        _session.delegate = self;
    }
    return _session;
}

- (void)startLive { 
    LFLiveStreamInfo *streamInfo = [LFLiveStreamInfo new];
    streamInfo.url = @"your server rtmp url";
    [self.session startLive:streamInfo];
}

- (void)stopLive {
    [self.session stopLive];
}

//MARK: - CallBack:
- (void)liveSession:(nullable LFLiveSession *)session liveStateDidChange: (LFLiveState)state;
- (void)liveSession:(nullable LFLiveSession *)session debugInfo:(nullable LFLiveDebug*)debugInfo;
- (void)liveSession:(nullable LFLiveSession*)session errorCode:(LFLiveSocketErrorCode)errorCode;
</code></pre>






