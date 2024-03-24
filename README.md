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
| Basler Pylon  | 7.4.0.14900   | 
| Gstreamer	    | 1.24.1        | 
| Camera        | daa3840-45uc  | 
| Camera        | acA2500-14gm  | 


*******************************************************************************************************

# Configuration for Windows 10 machine

## 1. Install Basler pylon software

https://www.baslerweb.com/en/products/software/basler-pylon-camera-software-suite/

Direct links:

Pylon 7.4.0 Camera Software Suite:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/basler_pylon_7_4_0_14900.exe

Pylon 7.4.0 Runtime:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/pylon_runtime_7_4_0_14900.exe

## 2. Test your setup with Pylon viewer

Open Pylon viewer. Search camera from devices list. Connect to camera, test it and configure features.

Finally Export features file.

Tools -> Save Features.

## 3. Install Gstreamer

https://gstreamer.freedesktop.org/download/

Direct links:

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/gstreamer-1.0-msvc-x86_64-1.24.1.msi

https://tequ-files.s3.eu.cloud-object-storage.appdomain.cloud/gstreamer-1.0-devel-msvc-x86_64-1.24.1.msi


ADD following Gstreamer path to 'PATH' environment variable. For example:
```
C:\gstreamer\1.0\msvc_x86_64\bin
```

## 4. Clone Basler gst-plugin-pylon

```
git clone https://github.com/basler/gst-plugin-pylon
```

## 5. Build and install gst-plugin-pylon 

Follow the instructions from Basler repository for building and installing gst-plugin-pylon. Instructions did not work as is 24.3.2024.

Here is short list of actions and commands, how building finally succeeded:

Install:
- Visual Studio Community Edition 2022 and C++ build tools
- meson 0.63.1-64 (https://github.com/mesonbuild/meson/releases)
- cmake 3.29 (https://cmake.org/)

Open command prompt and run commands:
```
cd\
```

```
md temp
```

```
cd temp
```

```
git clone https://github.com/basler/gst-plugin-pylon
```

```
cd gst-plugin-pylon
```

```
set PKG_CONFIG_PATH=%GSTREAMER_1_0_ROOT_MSVC_X86_64%lib\pkgconfig
```

```
set PATH=%PATH%;%GSTREAMER_1_0_ROOT_MSVC_X86_64%\bin
```

```
set CMAKE_PREFIX_PATH=C:\Program Files\Basler\pylon 7\Development\CMake\pylon\
```

```
meson setup build --prefix=%GSTREAMER_1_0_ROOT_MSVC_X86_64%
```

```
meson compile -C build
```

```
ninja -C build install
```

## 6. Pylon plugin for Gstreamer is ready to use


# Configuration for NVIDIA Jetson NX / Nano

Configuring Jetson for GStreamer with Basler cameras is bit different. Please see configuration steps at following repository:

https://github.com/Lapland-UAS-Tequ/tequ-jetson-basler 

# Configuration for Raspberry PI 4

Check this repository for Raspberry PI 4:

https://github.com/Lapland-UAS-Tequ/tequ-rpi-setup

# Example Gstreamer pipelines 

## Example 1. Show video from camera on screen. 

```
gst-launch-1.0 pylonsrc ! videoconvert ! autovideosink
```

## Example 2. Show video from camera on screen. 

Parameters supplied from configuration file and camera is selected using serial number. Configuration file can be generated using Basler Pylon viewer. 
```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! videoconvert ! autovideosink
```

## Example 3. Output JPEG stream to TCP server
```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! queue ! jpegenc ! tcpclientsink port=55555
```

## Example 4. Serve JPEG stream as TCP server

Connect with VLC tcp://localhost:8081

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! queue ! jpegenc ! tcpserversink port=55555
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
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! queue ! videoconvert ! queue ! x264enc tune=zerolatency ! rtspclientsink location=rtsp://localhost:8554/mystream
```

NVIDIA specific H264 encoder
```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! queue ! videoconvert ! queue ! nvh264enc ! rtspclientsink location=rtsp://localhost:8554/mystream
```

## Example 6. Use multiple sinks.

This pipeline publishes Basler camera video to TCP and RTSP and show on local screen. 

RTSP and TCP video streams are scaled to 1920x1080 from original 4k resolution.

Local screen video is 4k resolution.

```
gst-launch-1.0 pylonsrc pfs-location=40257292.pfs device-serial-number="40257292" ! queue ! bayer2rgb ! tee name=t t. ! queue ! videoscale ! video/x-raw,width=960,height=540 ! queue ! jpegenc ! queue ! tcpclientsink port=55555 t. ! queue ! videoscale ! video/x-raw,width=960,height=540 ! queue ! videoconvert ! queue ! nvh264enc ! queue ! rtspclientsink location=rtsp://localhost:8554/40122260 t. ! queue ! videoconvert ! queue ! autovideosink
```
