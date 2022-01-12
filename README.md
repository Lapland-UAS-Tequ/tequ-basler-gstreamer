# tequ-basler-gstreamer

This is guide how to install and configure necessary components to use Gstreamer to output video from Basler Camera.  

Tested with following setup:
| Component     | Version       |
| ------------- |:-------------:| 
| Windows       | 10            | 
| Basler Pylon  | 6.3.0         | 
| Gstreamer	    | 1.18.5        | 
| CMake         | 3.22.1        | 
| Visual Studio | 2022 Community edition | 
| Camera        | daa3840-45uc | 


Visual Studio 2022
 - At least component "C++ CMake Tools for Windows" and its dependencies must be installed


# Configuration for Windows 10 machine

## 1. Install Basler pylon software

https://www.baslerweb.com/en/products/software/basler-pylon-camera-software-suite/

## 2. Install Gstreamer

https://gstreamer.freedesktop.org/download/

ADD following Gstreamer paths to 'PATH' environment variable. For example:
```
C:\gstreamer\1.0\msvc_x86_64\bin
C:\gstreamer\1.0\msvc_x86_64\lib
C:\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0
```

Add system variable “GSTREAMER_DIR” with following path:

```
C:\gstreamer\1.0\x86_64
```

## 3. Clone gst-plugins-vision 

```
git clone https://github.com/Lapland-UAS-Tequ/gst-plugins-vision.git
```

## 4. Install Cmake 

https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-windows-x86_64.msi

1. Where is source code: "c:\gst-plugins-vision"

2. Where to build the binaries: ""c:\gst-plugins-vision\build"

3. Click "Configure" 

4. Check that "PYLON_DIR" is "C:/Program Files/Basler/pylon 6" and "PYLON_INCLUDE_DIR" is "C:/Program Files/Basler/pylon 6/Development/include"

5. Click "Generate"

6. Click "Open project"

7. Change "Debug" to "Release" 

8. From Solution Explorer right click "INSTALL" and select build 

Final output should be something like this:
```
...
...
...
10>------ Build started: Project: INSTALL, Configuration: Release x64 ------
10>-- Install configuration: "Release"
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstbayerutils.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstextractcolor.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstmisb.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstselect.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstvideoadjust.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstgentl.dll
10>-- Installing: C:/gstreamer/1.0/msvc_x86_64/lib/gstreamer-1.0/libgstpylon.dll
========== Build: 10 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

9. Pylon plugin for Gstreamer is ready to use

# Configuration for Linux machine

## TBD
## 1.
## 2.

# Example Gstreamer pipelines 

## Example 1. Show video from camera on screen. 

Parameters are supplied within pipeline command. Check https://github.com/Lapland-UAS-Tequ/gst-plugins-vision/blob/master/sys/pylon/gstpylonsrc.c for parameter names and use Pylon viewer to test and find out values for parameters. This example pipeline uses automatic values which work in many cases.
```
gst-launch-1.0 pylonsrc width=3840 height=2160 centerx=true centery=t acquisitionframerateenable=true fps=25 lightsource=5000k autoexposure=continuous exposurelowerlimit=250 exposureupperlimit=100000 autowhitebalance=continuous autogain=continuous gainupperlimit=40 gainlowerlimit=0 autobrightnesstarget=0.3 ! bayer2rgb ! videoconvert ! autovideosink
```

## Example 2. Show video from camera on screen. 

Parameters are supplied from configuration file. Configuration file can be generated using Basler Pylon viewer. 
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! bayer2rgb ! videoconvert ! autovideosink
```

## Example 3. Output JPEG stream to TCP server
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! bayer2rgb ! jpegenc ! queue ! tcpclientsink port=55555
```

