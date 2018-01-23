 全屏可以用flash画布舞台的stage属性值设置，不过要考虑下普通屏幕和宽屏的处理, 常见的显示器分辨率按其长宽比可分为为：4：3（1024×768）、5：4（1280×1024）、16：9、16：10 (这里暂以宽屏测试的，需要后续处理)，点击全屏按钮或点击视频画面都可以全屏，调用display()方法即可，代码片段如下：
  //切換全屏显示 
private function display():void
{ 
    if(!isFullScreen)
    { 
        stage.fullScreenSourceRect = new Rectangle(video.x, video.y,video.width,video.height);
        stage.displayState =StageDisplayState.FULL_SCREEN; 
        isFullScreen = true; 
    }else
    { 
        stage.displayState = StageDisplayState.NORMAL; 
        isFullScreen = false; 
    }
   
}