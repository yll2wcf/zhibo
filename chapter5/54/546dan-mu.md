##5.4.6弹幕
为了方便大家拓展功能，我们加入了一些简单的功能比如点赞、弹幕等。这里给大家介绍一个同样是由Bilibili站开源的DanmakuFlameMaster弹幕解析绘制框架。它也是目前为止Android上最好的弹幕框架，目前已经被优酷、土豆、斗鱼、ACFUN等APP软件使用。

####它的特性
* 使用多种方式(View/SurfaceView/TextureView)实现高效绘制
* B站xml弹幕格式解析
* 基础弹幕精确还原绘制
* 支持mode7特殊弹幕
* 多核机型优化，高效的预缓存机制
* 支持多种显示效果选项实时切换
* 实时弹幕显示支持
* 换行弹幕支持/运动弹幕支持
* 支持自定义字体
* 支持多种弹幕参数设置
* 支持多种方式的弹幕屏蔽

####集成方法
在Gradle中引入的方式
```code
repositories {
    jcenter()
}

dependencies {
    compile 'com.github.ctiao:DanmakuFlameMaster:0.9.21'
    compile 'com.github.ctiao:ndkbitmap-armv7a:0.9.21'

    # Other ABIs: optional
    compile 'com.github.ctiao:ndkbitmap-armv5:0.9.21'
    compile 'com.github.ctiao:ndkbitmap-x86:0.9.21'
}
```
在文件中加入弹幕控件
```xml
 <master.flame.danmaku.ui.widget.DanmakuView
        android:id="@+id/danmakuViewID"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
```
Activity中的功能代码也简单，注册弹幕控件并添加数据即可
```java
```