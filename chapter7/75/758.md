一般视频播放完毕后，播放的指针头归零，即播放进度条上的滑块指向起始位置，同时播放按钮状态为准备就绪状态，视频流处于暂停状态。可以通过视频流的当前状态信息进行判断，如下面e.info.code状态值可以获得各种不同的状态，这里只取播放完毕停止后的状态值，代码片段如下：
//播放完毕处理
private function NetStreamStatusHandler(e:NetStatusEvent):void
{
    if(e.info.code == "NetStream.Play.Stop")
    {
        ns.seek(0);
        btnPlay.source = playClass; 
        isPause = true;
        ns.pause();
    }
}