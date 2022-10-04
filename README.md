This repository is developed in Fish-IoT project

https://www.tequ.fi/en/project-bank/fish-iot/ 

---

# tequ-basler-gstreamer

This is guide how to install and configure necessary components to use Gstreamer to output video data from Basler Cameras. Examples are tested with Basler Dart daa3840-45uc with USB 3.0 interface and Basler ace acA2500-14gm camera with GigE interface.

Check these links for basics:

https://www.baslerweb.com/en/vision-campus/interfaces-and-standards/usb3/

https://www.baslerweb.com/en/vision-campus/interfaces-and-standards/gigabit-ethernet/

If you have problems connecting to cameras or keep losing fraimes, check that your hardware and cables are GigE or USB3 Vision compatible.

Tested with following setup:

| Component     | Version       |
| ------------- |:-------------:| 
| Windows       | 10            | 
| Basler Pylon  | 6.3.0         | 
| Gstreamer	    | 1.18.5        | 
| CMake         | 3.22.1        | 
| Visual Studio | 2022 Community edition | 
| Camera        | daa3840-45uc  | 
| Camera        | acA2500-14gm  | 

Visual Studio 2022
 - At least component "C++ CMake Tools for Windows" and its dependencies must be installed


*******************************************************************************************************

Basler has released official GStreamer Plug-in at 9th September 2022

https://www.baslerweb.com/en/company/news-press/news/pylon-gstreamer-plug-in-for-basler-cameras/877440/

https://github.com/basler/gst-plugin-pylon

*******************************************************************************************************


# Configuration for Windows 10 machine

## 1. Install Basler pylon software

https://www.baslerweb.com/en/products/software/basler-pylon-camera-software-suite/

Direct links:

Pylon 6.3.0 Camera Software Suite:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/Basler_pylon_6.3.0.23157.exe

Pylon 6.3.0 Runtime:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/pylon_Runtime_6.3.0.23157.exe

## 2. Test your setup with Pylon viewer

Open Pylon viewer. Search camera from devices list. Connect to camera, test it and configure features.

Finally Export features file.

Tools -> Save Features.

## 3. Install Gstreamer

https://gstreamer.freedesktop.org/download/

Direct links:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/gstreamer-1.0-msvc-x86_64-1.18.5.msi

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/gstreamer-1.0-devel-msvc-x86_64-1.18.5.msi


ADD following Gstreamer paths to 'PATH' environment variable. For example:
```
C:\gstreamer\1.0\msvc_x86_64\bin
C:\gstreamer\1.0\msvc_x86_64\lib
C:\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0
```

Add system variable “GSTREAMER_DIR” with following path:

```
C:\gstreamer\1.0\msvc_x86_64
```


## 4. Clone gst-plugins-vision 

```
git clone https://github.com/Lapland-UAS-Tequ/gst-plugins-vision.git
```

## 5. Build and install "gst-plugins-vision" with Cmake 

Download and install Cmake

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/cmake-3.22.1-windows-x86_64.msi

Open CMake application.

Configure, build and install "gst-plugins-vision"

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

# Configuration for NVIDIA Jetson NX / Nano

Configuring Jetson for GStreamer with Basler cameras is bit different. Please see configuration steps at following repository:

https://github.com/Lapland-UAS-Tequ/tequ-jetson-basler

# Configuration for Raspberry PI 4

Check this repository for Raspberry PI 4:

https://github.com/Lapland-UAS-Tequ/tequ-rpi-setup

# Example Gstreamer pipelines 

## Example 1. Show video from camera on screen. 

Parameters are supplied within pipeline command. Check https://github.com/Lapland-UAS-Tequ/gst-plugins-vision/blob/master/sys/pylon/gstpylonsrc.c for parameter names and use Pylon viewer to test and find out values for parameters. If you have YUV pixel format at camera, then you dont need to use "bayer2rgb". In any case you need to check that your camera supports parameters and their values you are providing. 

```
gst-launch-1.0 pylonsrc width=3840 height=2160 centerx=true centery=t acquisitionframerateenable=true fps=25 lightsource=5000k autoexposure=continuous exposurelowerlimit=250 exposureupperlimit=100000 autowhitebalance=continuous autogain=continuous gainupperlimit=40 gainlowerlimit=0 autobrightnesstarget=0.3 ! bayer2rgb ! videoconvert ! autovideosink
```

## Example 2. Show video from camera on screen. 

Parameters are supplied from configuration file. Configuration file can be generated using Basler Pylon viewer. 
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! queue ! bayer2rgb ! queue ! videoconvert ! autovideosink
```

## Example 3. Output JPEG stream to TCP server
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! queue ! bayer2rgb ! queue ! jpegenc ! tcpclientsink port=55555
```

## Example 4. Serve JPEG stream as TCP server

Connect with VLC tcp://localhost:8081

```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! queue ! bayer2rgb ! queue ! jpegenc ! tcpserversink port=55555
```

## Example 5. Publish video to simple-rtsp-server

Download extract and start rtsp-simple-server.

https://github.com/aler9/rtsp-simple-server

Direct download link:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/rtsp-simple-server.7z

Connect to stream with VLC 

rtsp://localhost:8554/mystream


Basic H264 encoder
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! queue ! bayer2rgb ! queue ! videoconvert ! queue ! x264enc tune=zerolatency ! rtspclientsink location=rtsp://localhost:8554/mystream
```

NVIDIA specific H264 encoder
```
gst-launch-1.0 pylonsrc config-file=40122260.pfs ! queue ! bayer2rgb ! queue ! videoconvert ! queue ! nvh264enc ! rtspclientsink location=rtsp://localhost:8554/mystream
```

## Example 6. Use multiple sinks.

This pipeline publishes Basler camera video to TCP and RTSP and show on local screen. 

RTSP and TCP video streams are scaled to 1920x1080 from original 4k resolution.

Local screen video is 4k resolution.

```
gst-launch-1.0 pylonsrc config-file=C:\\Users\\juha.autioniemi\\Desktop\\svn\\tequ\\dev\\Python\\apps\\tequ-basler-app\\configurations\\40122260.pfs ! queue ! bayer2rgb ! tee name=t t. ! queue ! videoscale ! video/x-raw,width=960,height=540 ! queue ! jpegenc ! queue ! tcpclientsink port=55555 t. ! queue ! videoscale ! video/x-raw,width=960,height=540 ! queue ! videoconvert ! queue ! nvh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/40122260 t. ! queue ! videoconvert ! queue ! autovideosink
```

## Example 7. Show video on Ubuntu desktop in Jetson. (YUV pixel format)

```
gst-launch-1.0 pylonsrc ! queue ! nvvidconv ! xvimagesink
```

## Example 8. Show video on Ubuntu desktop in Jetson. (bayer pixel format)

```
gst-launch-1.0 pylonsrc ! queue ! bayer2rgb ! queue ! nvvidconv ! xvimagesink
```

## Example 9. Convert video stream from camera to JPEG image stream to TCP port 55555. (YUV)
```
gst-launch-1.0 pylonsrc config-file=config.pfs ! queue ! nvvidconv ! nvjpegenc ! queue ! tcpclientsink port=55555
```

# Notes

If you are missing some libraries, here are some ways to install them:

bad-plugins 
```
sudo apt-get install gstreamer1.0-plugins-bad
```

RTSP
```
sudo apt-get install gstreamer1.0-rtsp
```
