##5.4.3滤镜
在开始介绍示例前，我们先给大家介绍一下滤镜的功能。我们平时看到直播里的主播都非常漂亮、皮肤也非常好，其实大部分都是滤镜美化的功劳。直播软件和我们平时用的修图工具一样，它也可以有磨皮、美白、增减亮度、锐化等功能。直播中实现滤镜功能的话，我们用的是之前介绍的Yasea框架中依赖的MagicCamera库，MagicCamera是一个支持实时滤镜的相机和视频录像的组件，包含美颜等40余种实时滤镜相机、可拍照、录像、图片修改等功能。GitHub地址是https://github.com/wuhaoyu1990/MagicCamera ，下面我们简单给大家介绍一下MagicCamera的使用方法。
### 一、相机预览模式
相机实时预览功能，我们只需要在程序的布局文件中集成MagicCameraView组件，代码如下：

```xml
<com.seu.magicfilter.widget.MagicCameraView
            android:id="@+id/glsurfaceview_camera"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

```
在Activity代码中注册MagicCameraView组件，并创建MagicEngine对象中创建，通过调用MagicEngine对象中的方法来对相机、滤镜进行操作。