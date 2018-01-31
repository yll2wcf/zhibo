###6.2.3 ijkplayer
ijkplayer 是一个基于 FFmpeg 的轻量级 Android/iOS 视频播放器。实现了跨平台功能，API易于集成；编译配置可裁剪，支持硬件加速解码，更加省电，目前比较火的美拍和斗鱼APP都在使用这个框架。

####ijkplayer的导入
下载ijkplayer 
在解压好的ijkplayer中，找到IJKMediaDemo。




打开IJKMediaDemo，编译
会提示'libavformat/avformat.h' file not found的错误，这是因为libavformat是ffmpeg中的库，而ijkplayer是基于ffmpeg这个库的，因此需要导入ffmpeg。

使用终端，cd 到ijkplayer-master目录中，输入./init-ios.sh运行脚本文件，init-ios.sh脚本的作用：下载ffmpeg源码，如果在前期准备中已经编译了ffmpeg，可以直接到ffmpeg中导入需要的文件。
执行完脚本后，就会发现ijkplayer中有ffmpeg了，但是这仅仅是把ffmpeg下载下来了，并没有编译，我们还要继续编译。
编译需要在终端输入如下命令：

```
cd ios
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```
./compile-ffmpeg.sh clean 是删除一些文件和文件夹，为编译ffmpeg.sh做准备。
./compile-ffmpeg.sh all 是真正的编译各个平台的ffmpeg库，并生成所以平台的通用库.
这是我们的ffmpeg已经彻底编译成功，如下图所示：



