#  		 [pkg-config 详解](https://www.cnblogs.com/rainsoul/p/10567390.html) 		

转载自：https://blog.csdn.net/newchenxf/article/details/51750239

1 什么是pkg-config

pkg-config是一个linux下的命令，用于获得某一个库/模块的所有编译相关的信息。
例子：

  pkg-config opencv –libs –cflags

结果：

-I/usr/include/opencv

/usr/lib/x86_64-linux-gnu/libopencv_calib3d.so /usr/lib/x86_64-linux-gnu/libopencv_contrib.so  /usr/lib/x86_64-linux-gnu/libopencv_core.so  /usr/lib/x86_64-linux-gnu/libopencv_features2d.so  /usr/lib/x86_64-linux-gnu/libopencv_flann.so  /usr/lib/x86_64-linux-gnu/libopencv_gpu.so  /usr/lib/x86_64-linux-gnu/libopencv_highgui.so  /usr/lib/x86_64-linux-gnu/libopencv_imgproc.so  /usr/lib/x86_64-linux-gnu/libopencv_legacy.so  /usr/lib/x86_64-linux-gnu/libopencv_ml.so  /usr/lib/x86_64-linux-gnu/libopencv_objdetect.so  /usr/lib/x86_64-linux-gnu/libopencv_ocl.so  /usr/lib/x86_64-linux-gnu/libopencv_photo.so  /usr/lib/x86_64-linux-gnu/libopencv_stitching.so  /usr/lib/x86_64-linux-gnu/libopencv_superres.so  /usr/lib/x86_64-linux-gnu/libopencv_ts.so  /usr/lib/x86_64-linux-gnu/libopencv_video.so  /usr/lib/x86_64-linux-gnu/libopencv_videostab.so

-lopencv_calib3d -lopencv_contrib -lopencv_core -lopencv_features2d -lopencv_flann  -lopencv_gpu -lopencv_highgui -lopencv_imgproc -lopencv_legacy  -lopencv_ml -lopencv_objdetect -lopencv_ocl -lopencv_photo  -lopencv_stitching -lopencv_superres -lopencv_ts -lopencv_video  -lopencv_videostab

  1
  2
  3
  4
  5

2 为什么要有pkg-config

从上面的例子，可以看出，pkg-config给出了opencv的头文件和库的所有信息！
这有什么好处？

所有用opencv的其他程序，在编译时，只需要写“pkg-config opencv –libs –cflags”,而不需要自己去找opencv的头文件在哪里，要链接的库在哪里！省时省力！
如果你写了一个库，不管是静态的还是动态的，要提供给第三方使用，那除了给人家库/头文件，最好也写一个pc文件，这样别人使用就方便很多，不用自己再手动写依赖了你哪些库，只需要敲一个”pkg-config [YOUR_LIB] –libs –cflags”。
3 pkg-config的信息从哪里来？

很简单，有2种路径：
第一种：取系统的/usr/lib下的所有*.pc文件。
第二种：PKG_CONFIG_PATH环境变量所指向的路径下的所有*.pc文件。

这些pc文件什么时候有的？都是在你安装某个库/模块的时候，添加的。比如你往系统安装opencv时，就会在/usr/lib/目录下，放一个opencv.pc。
比如，我的PC是这样的。你可以看到，有各种各样的pc文件。

chenxf@chenxf-PC:/usr/lib/x86_64-linux-gnu/pkgconfig$ ls
alsa.pc cairo-tee.pc fontenc.pc gl.pc harfbuzz.pc libdrm_radeon.pc
libswscale.pc pixman-1.pc x11-xcb.pc xcb-sync.pc xkbcommon.pc xvmc.pc
opencv.pc cairo-xcb.pc freetype2.pc glu.pc ice.pc libfs.pc 
libdrm.pc libraw1394.pc pciaccess.pc xcb-shm.pc
......

  1
  2
  3
  4
  5
  6

那么，pc文件到底写了什么？
打开看看就知道啦。比如opencv.pc。

\# Package Information for pkg-config
prefix=/usr
exec_prefix=${prefix}
libdir=${prefix}/lib/x86_64-linux-gnu
includedir_old=${prefix}/include/opencv
includedir_new=${prefix}/include

Name: OpenCV
Description: Open Source Computer Vision Library
Version: 2.4.8
Libs: -L${libdir} ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_calib3d.so  -lopencv_calib3d  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_contrib.so  -lopencv_contrib ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_core.so  -lopencv_core  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_features2d.so  -lopencv_features2d  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_flann.so -lopencv_flann  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_gpu.so -lopencv_gpu  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_highgui.so  -lopencv_highgui  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_imgproc.so  -lopencv_imgproc ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_legacy.so -lopencv_legacy ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_ml.so  -lopencv_ml ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_objdetect.so  -lopencv_objdetect ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_ocl.so  -lopencv_ocl ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_photo.so  -lopencv_photo  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_stitching.so  -lopencv_stitching  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_superres.so  -lopencv_superres ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_ts.so  -lopencv_ts ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_video.so  -lopencv_video  ${exec_prefix}/lib/x86_64-linux-gnu/libopencv_videostab.so  -lopencv_videostab
Cflags: -I${includedir_old} -I${includedir_new}

  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  11
  12

一目了然，就是存了所有opencv的头文件/库的路径信息。和第一步我们敲的”pkg-config opencv –libs –cflags”的结果，是一样一样的~~~
4 pkg-config都有哪些命令参数

所有参数，可以通过pkg-config –help来查看。
但我觉得其实就3个参数有用。
4.1 “pkg-config [NAME] –cflags”，查看头文件信息。

(注意这里是2个“-”，不知道为啥我原文写2个“-”，生成文章时只有一个。。。囧。。。不管了，反正提醒你一下啦)
例子：

  chenxf@chenxf-PC:~$ pkg-config opencv –cflags

结果：

-I/usr/include/opencv

  1

4.2 pkg-config [NAME] –libs，查看库信息。

例子：

  chenxf@chenxf-PC:~$ pkg-config opencv –libs

结果：

/usr/lib/x86_64-linux-gnu/libopencv_calib3d.so /usr/lib/x86_64-linux-gnu/libopencv_contrib.so  /usr/lib/x86_64-linux-gnu/libopencv_core.so  /usr/lib/x86_64-linux-gnu/libopencv_features2d.so  /usr/lib/x86_64-linux-gnu/libopencv_flann.so  /usr/lib/x86_64-linux-gnu/libopencv_gpu.so  /usr/lib/x86_64-linux-gnu/libopencv_highgui.so  /usr/lib/x86_64-linux-gnu/libopencv_imgproc.so  /usr/lib/x86_64-linux-gnu/libopencv_legacy.so  /usr/lib/x86_64-linux-gnu/libopencv_ml.so  /usr/lib/x86_64-linux-gnu/libopencv_objdetect.so  /usr/lib/x86_64-linux-gnu/libopencv_ocl.so  /usr/lib/x86_64-linux-gnu/libopencv_photo.so  /usr/lib/x86_64-linux-gnu/libopencv_stitching.so  /usr/lib/x86_64-linux-gnu/libopencv_superres.so  /usr/lib/x86_64-linux-gnu/libopencv_ts.so  /usr/lib/x86_64-linux-gnu/libopencv_video.so  /usr/lib/x86_64-linux-gnu/libopencv_videostab.so -lopencv_calib3d  -lopencv_contrib -lopencv_core -lopencv_features2d -lopencv_flann  -lopencv_gpu -lopencv_highgui -lopencv_imgproc -lopencv_legacy  -lopencv_ml -lopencv_objdetect -lopencv_ocl -lopencv_photo  -lopencv_stitching -lopencv_superres -lopencv_ts -lopencv_video  -lopencv_videostab

  1

4.3 pkg-config –list-all。查看pkg-config的所有模块信息。

例子：

  pkg-config –list-all

结果：

libass             libass - LibASS is an SSA/ASS subtitles rendering library
xext              Xext - Misc X Extension Library
pthread-stubs         pthread stubs - Stubs missing from libc for standard pthread functions
videoproto           VideoProto - Video extension headers
libxdot            libxdot - Library for parsing graphs in xdot format
gtk+-unix-print-2.0      GTK+ - GTK+ Unix print support
opencv             OpenCV - Open Source Computer Vision Library
xcb-randr           XCB RandR - XCB RandR Extension
renderproto          RenderProto - Render extension headers
xcb-present          XCB Present - XCB Present Extension
gdk-2.0            GDK - GTK+ Drawing Kit (x11 target)
libavfilter          libavfilter - FFmpeg audio/video filtering library
glesv2             glesv2 - Mesa OpenGL ES 2.0 library
xcursor            Xcursor - X Cursor Library
libavcodec           libavcodec - FFmpeg codec library
xi               Xi - X Input Extension Library
xrandr             Xrandr - X RandR Library

  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  11
  12
  13
  14
  15
  16
  17

5 如何添加自己的pc文件

如上文所说，有2种方式。
\1. 把你的pc文件，直接放到/usr/lib/…默认路径下。
\2. 把你的pc文件的路径写到PKG_CONFIG_PATH环境变量里。
比如，你可以在/etc/.bashrc或者/home/chenxf/.bashrc的文件末尾添加：

  PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/chenxf/ffmpeg_build/lib/pkgconfig
  export PKG_CONFIG_PATH

那么，pkg-config就会到/home/chenxf/ffmpeg_build/lib/pkgconfig寻找*.pc文件。

我猜你想问，那我这个pc文件何时生效呢？
答案是，如果是/usr/lib下，立马生效！！！如果在环境变量里，只要先source ~/.bashrc一下，让环境变量生成，也立马生效。
并不需要什么pkg-config update啥命令，让其更新信息。
其实每次你执行pkg-config，都会去遍历所有的*.pc文件。
6 如何自己写pkg-config的pc文件

其实很简单，只需要拿别人的pc文件改一改就行了。
不过我还是提一下吧。
pc文件的所有参数：

Name: 该模块的名字，比如你的pc名字是xxxx.pc，那么名字最好也是xxxx。
Description: 模块的简单描述。上文pkg-config –list-all命令出来的结果，每个名字后面就是description。
URL: 用户可以通过该URL获得更多信息，或者下载信息。也是辅助的，可要可不要。
Version: 版本号。
Requires: 该模块有木有依赖于其他模块。一般没有。
Requires.private: 该模块有木有依赖于其他模块，并且还不需要第三方知道的。一般也没有。
Conflicts: 有没有和别的模块冲突。常用于版本冲突。比如，Conflicts: bar < 1.2.3，表示和bar模块的1.2.3以下的版本有冲突。
Cflags: 这个就很重要了。pkg-config的参数–cflags就指向这里。主要用于写本模块的头文件的路径。
Libs: 也很重要，pkg-config的参数–libs就指向这里。主要用于写本模块的库/依赖库的路径。
Libs.private: 本模块依赖的库，但不需要第三方知道。

其实必须写的只有5个。Name、Description、Version、Cflags、Libs。

我们举2个例子吧。一个动态库，一个静态库。
例子1 动态库的pc文件

假设我写了libfoo.so，我的库将会被安装到/usr/local/lib/，头文件会放到/usr/local/include/foo。那么，pc文件可以这么写。

prefix=/usr/local
exec_prefix=${prefix}
includedir=${prefix}/include
libdir=${exec_prefix}/lib

Name: foo
Description: The foo library
Version: 1.0.0
Cflags: -I${includedir}/foo
Libs: -L${libdir} -lfoo

  1
  2
  3
  4
  5
  6
  7
  8
  9
  10

例子2 静态库的pc文件

正如我的另一博文所说，静态库链接动态库时，如何使用该静态库，如果我有个静态库libXXX.a，它依赖了很多其他动态库libAA.so，libBB.so，那么第三方程序DD.c要使用libXXX.a时，编译时还得链接libAA.so，libBB.so。
如何让第三方程序，可以不用操心我这个libXXX.a到底依赖了什么？很简答，就是我的libXXX.a写一个pc文件。

其实这个问题，就发生在我最近搞的ffmpeg事情上。
我下载了ffmpg，按照guide编译好了。生成了ffmpeg，以及相关的库。

chenxf@chenxf-PC:~/ffmpeg_build/lib$ ls
libavcodec.a  libavdevice.a libavfilter.a libavformat.a libavutil.a  libpostproc.a  libswresample.a libswscale.a libx264.a libx265.a  pkgconfig

  1
  2

当我写了个程序，想使用libavcodec.a，却各种编译错误。

  gcc test.c -o test -I/home/chenxf/ffmpeg_build/include/ -L/home/chenxf/ffmpeg_build/lib -lavcodec


/home/chenxf/ffmpeg_sources/ffmpeg/libavcodec/aacdec_template.c:1117：对‘pthread_once’未定义的引用
/home/chenxf/ffmpeg_build/lib/libavcodec.a(aacdec_fixed.o)：在函数‘aac_decode_init’中：
/home/chenxf/ffmpeg_sources/ffmpeg/libavcodec/aacdec_template.c:1117：对‘pthread_once’未定义的引用
/home/chenxf/ffmpeg_build/lib/libavcodec.a(aacenc.o)：在函数‘aac_encode_init’中：
/home/chenxf/ffmpeg_sources/ffmpeg/libavcodec/aacenc.c:1039：对‘pthread_once’未定义的引用

我当时还百思不得其解，为何用编译好了的libavcodec.a，第三方的程序，还会跟什么pthread有关！！！
答案是，第三方程序，还要链接libpthread.so。
那我如果编译第三方程序，还得一个个添加libavcodec.a锁依赖的库，岂不是累死了？！！！！

而其实，ffmpeg已经操心过这个事情了，它编译好了后，也生成了pc文件。就在“/home/chenxf/ffmpeg_build/lib/pkgconfig”

  chenxf@chenxf-PC:~/ffmpeg_build/lib/pkgconfig$ ls
   libavcodec.pc libavdevice.pc libavfilter.pc libavformat.pc libavutil.pc libpostproc.pc libswresample.pc libswscale.pc x264.pc x265.pc

随便打开一个看看。
libavcodec.pc

prefix=/home/chenxf/ffmpeg_build
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include

Name: libavcodec
Description: FFmpeg codec library
Version: 57.38.100
Requires: libswresample >= 2.0.101, libavutil >= 55.22.101
Requires.private:
Conflicts:
Libs: -L${libdir} -lavcodec -lXv -lX11 -lXext -lxcb -lXau -lXdmcp -lxcb-shm  -lxcb -lXau -lXdmcp -lxcb-xfixes -lxcb-render -lxcb-shape -lxcb -lXau  -lXdmcp -lxcb-shape -lxcb -lXau -lXdmcp -lX11 -lasound  -L/home/chenxf/ffmpeg_build/lib -lx265 -lstdc++ -lm -lrt -ldl  -L/home/chenxf/ffmpeg_build/lib -lx264 -lpthread -lm -ldl -lfreetype -lz -lpng12 -lass -lfontconfig -lenca -lm -lfribidi -lexpat -lfreetype -lz  -lpng12 -lm -llzma -lz -pthread
Libs.private:
Cflags: -I${includedir}

  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  11
  12
  13
  14
  15

看到没，看到没？！这里已经把libavcodec.a的所有依赖库，全部列出来了！！！！
所以，我唯一要做的，就是把这些pc文件的路径，写到PKG_CONFIG_PATH。
我就在/home/chenxf/.bashrc末行添加了

  PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/home/chenxf/ffmpeg_build/lib/pkgconfig
  export PKG_CONFIG_PATH

然后

  source /home/chenxf/.bashrc

然后重新编译我的代码，就通过了！！！！好开森的样子！！！！…………^^

  gcc test.c -o test pkg-config libavcodec libavformat libavutil --cflags --libs

清爽的编译命令，就能让我用尽所有ffmpeg的所有库，多好啊！感谢发明pkg-config的人！！！ 
\---------------------  