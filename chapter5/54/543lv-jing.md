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
```java
public class CameraActivity extends Activity{
    private MagicEngine magicEngine;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camera);
        MagicEngine.Builder builder = new MagicEngine.Builder();
        magicEngine = builder.build((MagicCameraView)findViewById(R.id.glsurfaceview_camera));
    }
}
```
设置滤镜方法
```java
//参数如下：MagicFilterType.EARLYBIRD
public void setCamerFilter(int filterType){
    magicEngine.setFilter(filterType);
}
```
设置美颜强度
```java
public void setCamerBeautyLevel(int level){
    magicEngine.setBeautyLevel(level);
}
```
保存图片方法
```java
public void setSavepicture(File file, SavePictureTask.OnPictureSaveListener listener){
    magicEngine.savePicture(file, listener);
}
```

设置切换前后摄像头
```java
public void setSwitchCamera(){
    magicEngine.switchCamera();
}
```

开启/关闭录制视频
```java
//开启录制视频；如果没有设置视频地址的话，默认保存地址是SD卡根目录
public void setStartRecord(){
    magicEngine.startRecord();
}
//关闭录制视频
public void setStopRecord(){
    magicEngine.stopRecord();
}
```
MagicCamera部分滤镜与Instagram中样式对照表如下：

MagicCamera| Instagram
---|---
MagicAmaroFilter | Instagram中Amaro滤镜
MagicAntiqueFilter | 复古滤镜
MagicBlackCatFilter | 黑猫滤镜，增强阴影与色调，颜色加深
MagicBrannanFilter | Instagram中Brannan滤镜
MagicBrooklynFilter | Instagram中Brooklyn滤镜
MagicCalmFilter | 平静滤镜，偏棕灰
MagicCoolFilter | 冰冷滤镜，偏蓝
MagicEarlyBirdFilter | Instagram中EarlyBird滤镜
MagicEmeraldFilter|祖母绿滤镜
MagicEvergreenFilter | 常青滤镜
MagicFairytaleFilter | 童话滤镜
MagicFreudFilter | Instagram中Freud滤镜
MagicHealthyFilter | 健康滤镜
MagicHefeFilter | Instagram中Hefe滤镜
MagicHudsonFilter | Instagram中Hudson滤镜
MagicInkwellFilter | Instagram中Inkwell滤镜
MagicKevinFilter | Instagram中Kevin滤镜
MagicLatteFilter | 拿铁滤镜
MagicLomoFilter | Instagram中Lomo滤镜
MagicN1977Filter | Instagram中N1977滤镜
MagicNashvilleFilter | Instagram中Nashville滤镜
MagicNostalgiaFilter | 怀旧滤镜，偏绿系
MagicPixarFilter | Instagram中Pixar滤镜
MagicRiseFilter | Instagram中Rise滤镜
MagicRomanceFilter | 浪漫滤镜，粉红系
MagicSakuraFilter | 樱花滤镜，粉红系
MagicSierraFilter | Instagram中Sierra滤镜
MagicSkinWhitenFilter | 美白滤镜，增白皮肤
MagicSunriseFilter | 日出滤镜
MagicSunsetFilter | 日落滤镜
MagicSutroFilter | Instagram中Sutro滤镜
MagicSweetsFilter | 甜美滤镜
MagicTenderFilter | 温和滤镜
MagicToasterFilter | Instagram中Toaster滤镜
MagicValenciaFilter | Instagram中Valencia滤镜
MagicWarmFilter | 温暖滤镜
MagicWhiteCatFilter | 白猫滤镜
MagicXproIIFilter | Instagram中XproII滤镜


