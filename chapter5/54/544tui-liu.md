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

```java
public class PushActivity extends AppCompatActivity implements RtmpHandler.RtmpListener,
        SrsRecordHandler.SrsRecordListener, SrsEncodeHandler.SrsEncodeListener{
 private SrsPublisher mPublisher;//推流对象
 private  ImageView btuTurnCameraID;
 
  @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_push_view);
        
       SrsCameraView srsCameraView = findViewById(R.id.pushViewCameraID);//相机组件
       btuTurnCameraID = findViewById(R.id.btuTurnCameraID);
       
       mPublisher = new SrsPublisher(srsCameraView);
        //设置编码状态的回调方法
        mPublisher.setEncodeHandler(new SrsEncodeHandler(this));
        //设置rtmp推流时状态回调
        mPublisher.setRtmpHandler(new RtmpHandler(this));
        //设置录像状态的回调方法
        mPublisher.setRecordHandler(new SrsRecordHandler(this));
        //设置预览分辨率
        mPublisher.setPreviewResolution(1920, 1080);
        //设置推流分辨率
        mPublisher.setOutputResolution(1080, 1920);
        //设置传输率
        mPublisher.setVideoHDMode();
        //设置直播源
        mPublisher.startPublish(urlStr);
        //开启摄像头
        mPublisher.startCamera();
        
         //切换前后摄像头
        btuTurnCameraID.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPublisher.switchCameraFace((mPublisher.getCamraId() + 1) % Camera.getNumberOfCameras());
            }
        });
    }
}
```