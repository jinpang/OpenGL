# OpenGL

一、流畅的播放逐帧动画
待选方案
1. 直接2D纹理逐帧加载PNG
2. 使用ETC压缩纹理替代PNG
3. 使用ETC2压缩纹理替代PNG
4. 使用PVRTC压缩纹理替代PNG
5. 使用S3TC压缩纹理替代PNG
文件大小对比
1. PNG图片大小与其内容有关，透明区域越多，大小越小。
2. ETC1图片每个像素占0.5byte,720*720png变为ETC后大小为720*720*2*0.5+16（alpha通道导致文件高度增加一倍，16个字节为文件头部信息），约507KBytes。
3. ETC2大小与设置相关，不包含A通道，大小与ETC1不保留A通道相同，包含A通道的，与ETC1保留A通道相同。
4. S3TC 相对于24位原图，DXT1压缩比例为6:1，DXT2-DXT5压缩比例为4:1。
5. PVRTC4 压缩比为6:1，PVRTC2压缩比为12:1（PVRTC图片宽高为2的幂数）
文件支持对比
1. PNG通用
2. ETC1是OpenGL2.0支持标准，基本上所有支持OpenGLES2.0，版本不低于2.2的Android设备都能使用。
3. ETC2是OpenGL3.0支持标准，基本上所有支持OpenGLES3.0，版本不低于4.3的Android设备都能使用。
4. S3TC广泛用于Windows平台上，DirectX中使用较多。在Android上支持率很低，主要是NVIDIA Tegra2芯片的手机。
5. PVRTC只有PowerVR的显卡支持。在苹果系中使用广泛。
方案1和方案2数据
针对测试用的60张png烟花图片动画进行量化分析（图片大小为720*720，手机360F4）：

PNG图片总大小为4.88M，ETC总大小29.6M。
PNG IO+解码耗时为15-40ms之间，与单张图片大小有关。ETC不在CPU中解码，只有IO时间，为4-10ms之间。（IO及解码时间与CPU能力及状态有关）
渲染时间二者基本一致。
针对方案2的补充方案

方案2文件总大小太大，针对这个问题，可采用zip压缩纹理，加载时直接加载zip中的纹理文件。数据如下：

总大小7.05M
IO+解码时间为4-16ms。
渲染时间同不进行压缩的ETC
注：不同手机不同环境时间数据不同，此数据仅为PNG加载和压缩纹理方式加载的对比。

from https://www.jianshu.com/p/299788c8a0c3

二、Android上用ETC1格式进行纹理压缩
from https://developer.android.com/studio/command-line/etc1tool
from http://blog.soliloquize.org/2014/12/12/Android%E4%B8%8A%E7%94%A8ETC1%E6%A0%BC%E5%BC%8F%E8%BF%9B%E8%A1%8C%E7%BA%B9%E7%90%86%E5%8E%8B%E7%BC%A9/

三、《OpenGL ES 3.0编程指南》中对可编程管线的流程介绍如下：
1. VBO/VAO（顶点缓冲区对象或顶点数组对象）
2. VertexShader（顶点着色器）
3. rasterization（光栅化）
4. FragmentShader（片段着色器）
5. Per-Fragment Operations（逐片段操作）
(1)pixelOwnershipTest（像素归属测试）
(2)ScissorTest（剪裁测试）
(3)StencilTest and DepthTest（模板和深度测试）
(4)Blending（混合）
(5)dithering（抖动）
6. Frame Buffer (帧缓冲区)
四、OpenGL ES与 EGL
什么是 OpenGL ES？
OpenGL ES（OpenGL for Embedded Systems）是 OpenGL 三维图形API的子集，针对手机、PDA和游戏主机等嵌入式设备而设计，各显卡制造商和系统制造商来实现这组 API。
EGL
什么是什么是 EGL？
EGL 是 OpenGL ES 渲染 API 和本地窗口系统(native platform window system)之间的一个中间接口层，它主要由系统制造商实现。
引入EGL就是为了屏蔽不同平台上的区别。
EGL提供如下的机制:
    与设备的原生窗口系统通信
    查询绘图表面的可用类型和配置
    创建绘图表面
    在OpenGL ES 和其他图形渲染API之间同步渲染
    管理纹理贴图等渲染资源
为了让OpenGL ES能够绘制在当前设备上，我们需要EGL作为OpenGL ES与设备的桥梁。
使用 EGL 绘图的基本步骤
    Display(EGLDisplay) 是对实际显示设备的抽象。
    Surface（EGLSurface）是对用来存储图像的内存区域
    FrameBuffer 的抽象，包括 Color Buffer， Stencil Buffer ，Depth Buffer。Context (EGLContext) 存储 OpenGL ES绘图的一些状态信息。
    
使用EGL的绘图的一般步骤：
    获取 EGL Display 对象：eglGetDisplay()
    初始化与 EGLDisplay 之间的连接：eglInitialize()
    获取 EGLConfig 对象：eglChooseConfig()
    创建 EGLContext 实例：eglCreateContext()
    创建 EGLSurface 实例：eglCreateWindowSurface()
    连接 EGLContext 和 EGLSurface：eglMakeCurrent()
    使用 OpenGL ES API 绘制图形：gl_*()
    切换 front buffer 和 back buffer 送显：eglSwapBuffer()
    断开并释放与 EGLSurface 关联的 EGLContext 对象：eglRelease()
    删除 EGLSurface 对象
    删除 EGLContext 对象
    终止与 EGLDisplay 之间的连接
    
OpenGL ES 和 EGL 的联系
OpenGL ES 本质上是一个图形渲染管线的状态机
而 EGL 则是用于监控这些状态以及维护 Frame buffer 和其他渲染 Surface 的外部层。
下图是一个3d游戏典型的 EGL 系统布局图。
from:https://blog.csdn.net/cauchyweierstrass/article/details/53189449

