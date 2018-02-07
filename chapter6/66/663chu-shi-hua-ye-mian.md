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
6.3节中我们讲过GPUImage是基于OpenGLES的，而使用OpenGLES程序来处理图片，一般会有4个步骤：
1.初始化OpenGL ES环境，编译、链接顶点着色器和片元着色器；
2、缓存顶点、纹理坐标数据，传送图像数据到GPU；
3、绘制图元到特定的帧缓存；
4、在帧缓存取出绘制的图像。

