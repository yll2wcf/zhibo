##6.6.3滤镜
在开启直播前，我们通常会对采集到的视频做一些美化处理，使视频看上去更加美观，让人物更加上镜。常用的一些处理包括磨皮、美白、提高亮度、提高饱和度等，这些操作都是滤镜其中的一种。滤镜处理的原理:就是把静态图片或者视频的每一帧进行图形变换再显示出来。它的本质就是像素点的坐标和颜色变化。我们要实现滤镜功能，就需要使用GPUImage这个开源框架，也可以直接使用LFLiveKit框架，LFLiveKit内部已经对GPUImage做了一次简单封装。本节我们会分别介绍这两种框架在视频美化上的使用。
###LFLiveKit美颜
LFLiveKit框架中LFGPUImageBeautyFilter就是已经封装好GPUImage的类，我们可以直接调用里面的方法进行美颜。
这个类给我们提供了3个方法分别实现美白调节、亮度调节、色调调节，代码如下：


```
- (id)init;
{
    if (!(self = [super initWithFragmentShaderFromString:kLFGPUImageBeautyFragmentShaderString])) {
        return nil;
    }
    //设置色彩饱和度
    _toneLevel = 0.5;
    //设置美白度
    _beautyLevel = 0.5;
    //设置亮度
    _brightLevel = 0.5;
    [self setParams:_beautyLevel tone:_toneLevel];
    [self setBrightLevel:_brightLevel];
    return self;
}

- (void)setInputSize:(CGSize)newSize atIndex:(NSInteger)textureIndex {
    [super setInputSize:newSize atIndex:textureIndex];
    inputTextureSize = newSize;

    CGPoint offset = CGPointMake(2.0f / inputTextureSize.width, 2.0 / inputTextureSize.height);
    [self setPoint:offset forUniformName:@"singleStepOffset"];
}
//调节美颜等级
- (void)setBeautyLevel:(CGFloat)beautyLevel {
    _beautyLevel = beautyLevel;
    [self setParams:_beautyLevel tone:_toneLevel];
}
//调节亮度
- (void)setBrightLevel:(CGFloat)brightLevel {
    _brightLevel = brightLevel;
    [self setFloat:0.6 * (-0.5 + brightLevel) forUniformName:@"brightness"];
}

- (void)setParams:(CGFloat)beauty tone:(CGFloat)tone {
    GPUVector4 fBeautyParam;
    fBeautyParam.one = 1.0 - 0.6 * beauty;
    fBeautyParam.two = 1.0 - 0.3 * beauty;
    fBeautyParam.three = 0.1 + 0.3 * tone;
    fBeautyParam.four = 0.1 + 0.3 * tone;
    [self setFloatVec4:fBeautyParam forUniform:@"params"];
}
```
我们使用起来非常方便，直接调用即可，代码如下：


```
    //开启或关闭美颜
    self.session.beautyFace=YES;
    //设置美颜程度
    self.session.beautyLevel=0.5;
    //设置亮度
    self.session.brightLevel=0.5;
```

###GPUImage美颜
在6.3节中我们讲过GPUImage是基于OpenGLES的，而使用OpenGLES程序来处理图片，一般会有4个步骤：
1. 初始化OpenGL ES环境，编译、链接顶点着色器和片元着色器；
2、缓存顶点、纹理坐标数据，传送图像数据到GPU；
3、绘制图元到特定的帧缓存；
4、在帧缓存取出绘制的图像。
在GPUImage框架中GPUImageFilter类主要负责上述的1-3步，GPUImageFramebufferz类主要负责第4步。了解了OpenGLES的处理过程，再来看看GPUImage框架处理过程，6.3节中讲过GPUImage是采用链式方法来处理画面，具体分为3个环节：source->filter->final target。
**1.source(视频、图片源)：**
GPUImageVideoCamera 摄像头用于实时拍摄视频GPUImageStillCamera 摄像头用于实时拍摄照片GPUImagePicture 用于处理已经拍摄好的图片GPUImageMovie 用于处理已经拍摄好的视频
**2.filter(滤镜)**
GPUImageFilter：就是用来接收源图像，通过自定义的顶点、片元着色器来渲染新的图像，并在绘制完成后通知响应链的下一个对象。GPUImageFramebuffer：就是用来管理纹理缓存的格式与读写帧缓存的buffer。
**3.final target(处理后的视频、图片)**
GPUImageView,GPUImageMovieWriter最终输入目标，响应链的终点，显示图片或者视频。

我们实现美颜主要是两个方面：磨皮、美白。
* 磨皮(GPUImageBilateralFilter)：本质就是让像素点模糊，可以使用高斯模糊，但是可能导致边缘会不清晰，用双边滤波(Bilateral Filter) ，有针对性的模糊像素点，能保证边缘不被模糊。
* 美白(GPUImageBrightnessFilter)：本质就是提高亮度。
这两个组合成滤镜组链使用，就会达到我们要的美颜效果。实现代码如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建视频源
    // SessionPreset:屏幕分辨率，AVCaptureSessionPresetHigh会自适应高分辨率
    // cameraPosition:摄像头方向
    GPUImageVideoCamera *videoCamera = [[GPUImageVideoCamera alloc] initWithSessionPreset:AVCaptureSessionPresetHigh cameraPosition:AVCaptureDevicePositionFront];
     videoCamera.outputImageOrientation = UIInterfaceOrientationPortrait;
    _videoCamera = videoCamera;

    // 创建最终输出View
    GPUImageView *captureVideoPreview = [[GPUImageView alloc] initWithFrame:self.view.bounds];
    [self.view insertSubview:captureVideoPreview atIndex:0];
    
    // 创建滤镜：磨皮和美白，这个就是我们组合滤镜，称为滤镜组，
    GPUImageFilterGroup *groupFilter = [[GPUImageFilterGroup alloc] init];
    
    // 磨皮滤镜
    GPUImageBilateralFilter *bilateralFilter = [[GPUImageBilateralFilter alloc] init];
    [groupFilter addTarget:bilateralFilter];
    _bilateralFilter = bilateralFilter;
    
    // 美白滤镜
    GPUImageBrightnessFilter *brightnessFilter = [[GPUImageBrightnessFilter alloc] init];
    [groupFilter addTarget:brightnessFilter];
    _brightnessFilter = brightnessFilter;
    
    // 设置滤镜组链
    [bilateralFilter addTarget:brightnessFilter];
    [groupFilter setInitialFilters:@[bilateralFilter]];
    groupFilter.terminalFilter = brightnessFilter;
    
    // 设置GPUImage响应链，从数据源 => 滤镜组 => 最终界面效果
    [videoCamera addTarget:groupFilter];
    [groupFilter addTarget:captureVideoPreview];
    
    // 必须调用startCameraCapture，底层才会把采集到的视频源，渲染到GPUImageView中，就可以显示了。
    // 开始采集视频
    [videoCamera startCameraCapture];
}

- (IBAction)brightnessFilter:(UISlider *)sender {
    _brightnessFilter.brightness = sender.value;
}

- (IBAction)bilateralFilter:(UISlider *)sender {
    // 值越小，磨皮效果越好
    CGFloat maxValue = 10;
    [_bilateralFilter setDistanceNormalizationFactor:(maxValue - sender.value)];
}
```
**注意**
* SessionPreset最好使用AVCaptureSessionPresetHigh，会自动识别，如果用太高分辨率，设备不支持时会直接报错
* GPUImageVideoCamera必须要强引用，否则会被销毁，不能持续采集视频.
* 必须调用startCameraCapture，底层才会把采集到的视频源，渲染到GPUImageView中。
* GPUImageBilateralFilter的distanceNormalizationFactor值越小，磨皮效果越好,distanceNormalizationFactor取值范围: 大于1。

**最后**
GPUImage是相当强大的开源框架，它能实现的效果远远比美颜效果多的多，它的源码也有很好的学习价值，
