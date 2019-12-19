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

一、Android上用ETC1格式进行纹理压缩
from https://developer.android.com/studio/command-line/etc1tool
from http://blog.soliloquize.org/2014/12/12/Android%E4%B8%8A%E7%94%A8ETC1%E6%A0%BC%E5%BC%8F%E8%BF%9B%E8%A1%8C%E7%BA%B9%E7%90%86%E5%8E%8B%E7%BC%A9/
