##5.4.2框架导入
示例项目中为了简洁、方便，除了Yasea框架，其他依赖我们都是在Gradle中引入的方式来引用的，项目结构如下图。


```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.FLASHLIGHT" />

    <uses-feature android:name="android.hardware.camera.autofocus" />
    <uses-feature android:glEsVersion="0x00020000" android:required="true" />
```