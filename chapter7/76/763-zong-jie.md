###7.6.3 总结
经过上文的介绍，我们大致了解了web端推流的方法，大致流程如下：
- 安装FlashBuilder并创建一个Flex项目
- 使用ActionScript编写一个推流器/拉流器
- 将Flex生成的swf播放器文件通过swfobject.js嵌入到HTML中
- 利用JavaScript与播放器的通信控制推流拉流与播放等等

我们在实际的使用中应该将常用的JavaScript功能封装成一套SDK以方便使用，下面是我封装的一些常用属性以及方法名称：


```
initPublish = {
                previewWindowWidth: 862,//视频预览宽度，默认862
                previewWindowHeight: 446,//视频预览高度，默认446
                videoWidth: 640,//视频推流分辨率宽
                videoHeight: 480,//视频推流分辨率高
                fps: 15,//帧数
                bitrate: 600,//比特率
                wmode: "transparent",//flash显示模式
                quality: "high",//flash显示质量
                allowScriptAccess: "always"//允许flash跨域
}

start = function() {...} //开始拉流
stop = function() {...} //停止拉流
startPublish = function() {...} //开始推流
stopPublish = function() {...} //停止推流
getUrl = function() {...} //获取流地址
getCamera = function(...) {...} //获取本地摄像头
getMicro = function() {...} //获取本地麦克风
getFlexCall = function() {...} //接受flex的消息
sentFlex = function() {...} //向flex发送消息
//...
                
```

实际的需求千变万化，我们在编写自己的SDK时要灵活多变，总结出一套最适合项目需求的SDK。
好了，web端的推拉流功能的实现就介绍到这里。