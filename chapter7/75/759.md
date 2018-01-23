视频声音控制通过SoundTransform 类操作，该类包含音量和平移的属性。如果禁音后运行调节滑块的话，需要再定义一个临时变量tmpSound，以便开启声音时为最终设置的音量。
 //静音、开音控制
private function closeSound():void
{ 
    if(isSound)
    {
        soundImg.source = sound; 
        tmpSound= ns.soundTransform;
        soundTrans.volume = 0;
        ns.soundTransform = soundTrans; // 这里禁音直接ns.soundTransform.volume = 0 这样不行，需要用对象赋值
        isSound = false; 
    }else
    { 
        soundImg.source = sound1; 
        ns.soundTransform = tmpSound; 
        isSound = true; 
    } 
}
   
//通过滑块调整声音 
private function sound_thumbChanges(event:SliderEvent):void
{ 
    tmpSound.volume = soundProcess.value;
    ns.soundTransform = tmpSound; 
} 