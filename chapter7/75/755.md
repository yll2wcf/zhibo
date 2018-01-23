//播放进度和缓冲进度处理
private function EnterFrameHandler(e:Event):void
{ 
   
    if (totalTime>0)
    { 
        playTime = ns.time;// ns.time为流媒体实时播放的时间
    }
   
    if (ns.bytesLoaded>0)
    { 
        bufferRect.width = ns.bytesLoaded / ns.bytesTotal*(playProcess.width-10);//计算缓冲方框的宽度(滑块本身也有一定的宽度,减去约10个像素宽度)
    }              
   
}  
说明：
   playTime作为播放进度条当前实时播放的时间点，视频的总时间作为播放显示进度条的最大值。
    <mx:HSlider id="playProcess"minimum="0" value="{playTime}" maximum="{totalTime}" 
   ns.bytesLoaded为已缓冲好的流媒体字节大小（单位为byte），ns.bytesTotal为流媒体的总大小，通过比例计算（如上）可以得出缓冲进度在播放进度条上的位置，缓冲进度条其实是一个长方形框，以层的形式位于播放进度条下放，初始宽度为0,当缓冲达到100%时，即缓冲完毕时，缓冲条的长度和播放进度条等长（除去滑块宽度）。缓冲方框可以是一个BorderContainer，如下：
    <s:BorderContainer x="14" y="411" width="0" height="4" id="bufferRect" buttonMode="true" borderColor="#70b2ee" backgroundColor="#70b2ee">
    </s:BorderContainer>    
页面所有的控件和标签如下：

    <mx:Image source="{playClass}" click="play();" id="btnPlay" buttonMode="true" x="16" y="423"/>  
    <mx:UIComponent id="uic" height="400" width="640" click="play()" doubleClickEnabled="true" doubleClick="display()" buttonMode="true" y="0" x="0"/> 
    <s:BorderContainer x="14" y="411" width="0" height="4" id="bufferRect" buttonMode="true" borderColor="#70b2ee" backgroundColor="#70b2ee">
    </s:BorderContainer>
    <mx:Label text="{formatTime(playTime)}/{formatTime(totalTime)}                    {fileSize}" width="150" x="60" y="427"/>
    <mx:HSlider id="playProcess"  minimum="0" value="{playTime}" maximum="{totalTime}" change="play_onchange(event)" thumbPress="thumbPress();"  thumbRelease="thumbRelease();"
                alpha="0.5"    dataTipFormatFunction="dataTipFormat" buttonMode="true" showTrackHighlight="true" x="9" y="398" width="630"/>
    
    <mx:Image source="{sound1}" click="closeSound();" id="soundImg" buttonMode="true" x="364" y="421"/>
    <mx:HSlider id="soundProcess" x="406" y="420"  minimum="0" maximum="1" change="sound_thumbChanges(event)"  
                showTrackHighlight="true" dataTipFormatFunction="SoundTipFormat"   buttonMode="true"  width="135"/>  
    <mx:Button label="全屏" click="display();" buttonMode="true" cornerRadius="20" labelPlacement="right" paddingLeft="6" x="561" y="423"/>
    <mx:SWFLoader id="load" x="16" y="486"/>