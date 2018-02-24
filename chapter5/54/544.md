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
在Activity里面进行组件注册、功能设置等，代码如下

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
在上面代码中可以看到，我们分别对编码、RTMP、录像设置了状态回调方法，我们可以在这些方法中获得是否连接成功、是否断开、视频码率(Bitrate), 帧率(FPS)状态等
```java
    //网络弱
    @Override
    public void onNetworkWeak() {}
    //网路重新连接
    @Override
    public void onNetworkResume() {}
    //编码出现IllegalArgumentException错误
    @Override
    public void onEncodeIllegalArgumentException(IllegalArgumentException e) {}
    //录像暂停
    @Override
    public void onRecordPause() {}
    //录像重新开始
    @Override
    public void onRecordResume() {}
    //录像开始
    @Override
    public void onRecordStarted(String msg) {
        Toast.makeText(getApplicationContext(), msg, Toast.LENGTH_SHORT).show();
    }
    //录像关闭
    @Override
    public void onRecordFinished(String msg) {
        Toast.makeText(getApplicationContext(), msg, Toast.LENGTH_SHORT).show();
    }
    //录像出现IllegalArgumentException错误
    @Override
    public void onRecordIllegalArgumentException(IllegalArgumentException e) {}
    //录像出现IOException错误
    @Override
    public void onRecordIOException(IOException e) {}
    //RTMP连接
    @Override
    public void onRtmpConnecting(String msg) {
        Toast.makeText(getApplicationContext(), "连接中。。。", Toast.LENGTH_SHORT).show();
    }
    //RTMP连接成功
    @Override
    public void onRtmpConnected(String msg) {
        Toast.makeText(getApplicationContext(), "连接成功，您的Show Time", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onRtmpVideoStreaming() {}

    @Override
    public void onRtmpAudioStreaming() {}

    //流媒体结束
    @Override
    public void onRtmpStopped() {}

    //流媒体连接
    @Override
    public void onRtmpDisconnected() {
        Toast.makeText(getApplicationContext(), "直播断开连接", Toast.LENGTH_SHORT).show();
    }

    //流媒体FPS改变时
    @Override
    public void onRtmpVideoFpsChanged(double fps) {
        Log.i(TAG, String.format("Output Fps: %f", fps));
    }
    //流媒体视频码率(Bitrate)状态改变时
    @Override
    public void onRtmpVideoBitrateChanged(double bitrate) {
        int rate = (int) bitrate;
        if (rate / 1000 > 0) {
            Log.i(TAG, String.format("Video bitrate: %f kbps", bitrate / 1000));
        } else {
            Log.i(TAG, String.format("Video bitrate: %d bps", rate));
        }
    }
    //音频视频码率(Bitrate)状态改变时
    @Override
    public void onRtmpAudioBitrateChanged(double bitrate) {
        int rate = (int) bitrate;
        if (rate / 1000 > 0) {
            Log.i(TAG, String.format("Audio bitrate: %f kbps", bitrate / 1000));
        } else {
            Log.i(TAG, String.format("Audio bitrate: %d bps", rate));
        }
    }
    //RTMP Socket错误
    @Override
    public void onRtmpSocketException(SocketException e) {}

    //RTMP IOException错误
    @Override
    public void onRtmpIOException(IOException e) {}

    @Override
    public void onRtmpIllegalArgumentException(IllegalArgumentException e) {}

    @Override
    public void onRtmpIllegalStateException(IllegalStateException e) {}
```