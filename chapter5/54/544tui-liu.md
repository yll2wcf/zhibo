##5.4.4推流
推流功能我们依赖的是Yasea框架，前面章节我们已经介绍过了，这里我们之间来看看如何运用到代码之中。先在我们项目中导入依赖库，然后在布局文件中引用相机组件，代码如下:

```xml
<net.ossrs.yasea.SrsCameraView
        android:id="@+id/pushViewCameraID"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_alignParentBottom="true"
        android:layout_alignParentEnd="true"
        android:layout_alignParentLeft="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true" />
```
