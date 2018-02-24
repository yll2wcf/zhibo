## 5.4.2框架导入

示例项目中为了简洁、方便，除了Yasea框架，其他依赖我们都是在Gradle中引入的方式来引用的，项目结构如下图。
* MainActivity 主界面
* PullActivity 拉流功能代码
* PushActivity 推流功能代码

![](/assets/图5.4.2-1.png)
#### 依赖引用说明

```code
    //Yesea推流框架引用
    implementation project(':YeseaLibrary')

    //权限管理
    implementation "io.reactivex.rxjava2:rxjava:2.1.9"
    implementation 'com.tbruyelle.rxpermissions2:rxpermissions:0.9.5@aar'

    //ijkplayer核心代码
    implementation 'tv.danmaku.ijk.media:ijkplayer-java:0.8.4'
    //ijkplayer不同CPU下引用
    implementation 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.4'
    implementation 'tv.danmaku.ijk.media:ijkplayer-armv5:0.8.4'
    implementation 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.4'
    implementation 'tv.danmaku.ijk.media:ijkplayer-x86:0.8.4'
    implementation 'tv.danmaku.ijk.media:ijkplayer-x86_64:0.8.4'

    //弹幕组件核心代码
    compile 'com.github.ctiao:DanmakuFlameMaster:0.9.21'
    //弹幕不同CPU下引用
    compile 'com.github.ctiao:ndkbitmap-armv7a:0.9.21'
    compile 'com.github.ctiao:ndkbitmap-armv5:0.9.21'
    compile 'com.github.ctiao:ndkbitmap-x86:0.9.21'
```

#### 所需权限

```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.FLASHLIGHT" />

    <uses-feature android:name="android.hardware.camera.autofocus" />
    <uses-feature android:glEsVersion="0x00020000" android:required="true" />
```



