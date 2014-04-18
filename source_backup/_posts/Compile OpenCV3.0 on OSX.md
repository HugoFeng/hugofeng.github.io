title: Compile OpenCV 3.0 on OSX
date: 2014-04-17 17:37:43
tags: [opencv]
---
## Install

Installing OpenCV on Mac can be fairly simple--using `homebrew`, however, `homebrew` can only install a stable version of it(currently 2.4.8.2). If you want to have a taste of OpenCV3.0, you'll have to build from the source. This sometimes can be very annoying[1][2]. After more than a day's struggle, I find a solution for my machine.

If you have anaconda installed, probably the vtk module has a version of 5.x, but OpenCV3.0 need vtk at least 6.1.[1]

{% codeblock lang:shell %}
$ brew install vtk   # This gonna take some time
{% endcodeblock %}

Now download the latest OpenCV source:
{% codeblock lang:shell %}
$ git clone git@github.com:Itseez/opencv.git
{% endcodeblock %}

Go to OpenCV source folder, 
{% codeblock lang:shell %}
$ mkdir build
$ cd build
$ cmake "Unix Makefile" -D CMAKE_OSX_ARCHITECTURES=x86_64 -D BUILD_PERF_TESTS=OFF ..
{% endcodeblock %}

Since OpenCV module ‘viz’ has to be compiled with libc++ instead of libstdc++, we need to make some changes to the makefiles to ensure it is compiled that way.
{% codeblock lang:shell %}
$ cd modules/viz/CMakeFiles/opencv_viz.dir/
$ vim flags.make
{% endcodeblock %}

Add flag `-std=c++11 -stdlib=libc++`
Do the same to build/modules/viz/CMakeFiles/opencv_test_viz.dir/flags.make
Now back to the build folder. 
{% codeblock lang:shell %}
$ make -j8  # Using 8 threads to build
{% endcodeblock %}

And it should work fine. 
Now finish up:
{% codeblock lang:shell %}
$ make install
{% endcodeblock %}

## Check

There're generally 2 ways to check OpenCV's version. 

1. In the OpenCV source folder. modules/core/include/opencv2/core/version.hpp 
2. After `make install`, you can see the copied libraries, they are usually named with version numbers. Like "libopencv_core.3.0.0.dylib".

Also, you can use `pkg-config` to check if the libs are correctly located.
{% codeblock lang:shell %}
$pkg-config --libs opencv
{% endcodeblock %}

The output will look like this:

	/usr/local/lib/libopencv_calib3d.dylib /usr/local/lib/libopencv_contrib.dylib /usr/local/lib/libopencv_core.dylib /usr/local/lib/libopencv_cuda.dylib /usr/local/lib/libopencv_cudaarithm.dylib /usr/local/lib/libopencv_cudabgsegm.dylib /usr/local/lib/libopencv_cudafeatures2d.dylib /usr/local/lib/libopencv_cudafilters.dylib /usr/local/lib/libopencv_cudaimgproc.dylib /usr/local/lib/libopencv_cudaoptflow.dylib /usr/local/lib/libopencv_cudastereo.dylib /usr/local/lib/libopencv_cudawarping.dylib /usr/local/lib/libopencv_features2d.dylib /usr/local/lib/libopencv_flann.dylib /usr/local/lib/libopencv_highgui.dylib /usr/local/lib/libopencv_imgproc.dylib /usr/local/lib/libopencv_legacy.dylib /usr/local/lib/libopencv_ml.dylib /usr/local/lib/libopencv_nonfree.dylib /usr/local/lib/libopencv_objdetect.dylib /usr/local/lib/libopencv_optim.dylib /usr/local/lib/libopencv_photo.dylib /usr/local/lib/libopencv_shape.dylib /usr/local/lib/libopencv_softcascade.dylib /usr/local/lib/libopencv_stitching.dylib /usr/local/lib/libopencv_superres.dylib /usr/local/lib/libopencv_ts.a /usr/local/lib/libopencv_video.dylib /usr/local/lib/libopencv_videostab.dylib /usr/local/lib/libopencv_viz.dylib


ref: 
[1] http://code.opencv.org/issues/3582
[2] http://stackoverflow.com/questions/19671827/opencv-installation-on-mac-os-x
[3] http://ibivanchev.blogspot.be/2013/10/opencv-mac-os-x-109.html

