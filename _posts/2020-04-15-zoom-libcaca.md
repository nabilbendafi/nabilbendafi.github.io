---
layout: post
title: Ascii art filter in Zoom
subtitle: It smells like poop.
tags: [video, zoom, linux, libcaca]
image: https://lh3.googleusercontent.com/proxy/37hNtZzCYk27EYDzoRsPiARLy05MSz27DQjFJFKhofy8PJIl1A_XHh1uDg65wdXSSqYv4Uy-Jz5xi6wmHX-MaRDjVA
share-img: https://lh3.googleusercontent.com/proxy/37hNtZzCYk27EYDzoRsPiARLy05MSz27DQjFJFKhofy8PJIl1A_XHh1uDg65wdXSSqYv4Uy-Jz5xi6wmHX-MaRDjVA
---

During [COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) confinement, I had to use [Zoom](https://zoom.us/) to my daily chats and meeting with colleagues.

I had a old version installed, and after updating to a newer version, I found that we can have _Virtual Background_ which is pretty funny, but Linux version (3.5.3), lacks for animated ones.

To had some extra fun during my calls, I wanted to integrate some funny filter, one might find in several video applications.

After several search, I've found that combining multiple tools and commands I was able to "integrate" ASCII art filter to my Linux Zoom setup.

# Tools
## Hasciicam

[hasciicam](https://ascii.dyne.org) is a project that converts video from TV card or webcam into ASCII. It uses [AAlib](http://aa-project.sourceforge.net/aalib) underneath to render YUV420 video stream into ASCII art.
It only output greyscale images, though.

## Libcaca
[libcaca](http://caca.zoy.org/wiki/libcaca) is a graphics library that outputs text instead of pixels, so that it can work on older video cards or text terminals. And its output is in <span style="color:red">c</span><span style="color:orange">o</span><span style="color:orange">l</span><span style="color:green">o</span><span style="color:blue">r</span>.
Its output is in [rgb24](http://avisynth.nl/index.php/RGB24).

## V4L2

[v4l2loopback](https://github.com/umlaeute/v4l2loopback) is kernel module that create dummy video you can interact with via v4l2 API.
```bash
sudo apt install v4l2loopback-utils v4l2loopback-dkms gstreamer1.0-tools
sudo modprobe v4l2loopback card_label="VirtualCam #0"
ls /dev/video1
```

## FFMPEG
[ffmpeg](https://www.ffmpeg.org) is a great tool to manipulate audio and video.

Assuming your _real_ webcam is attached and accessible via `/dev/video0`, you can simply "copy" its video stream and make it available through the _dummy_ device provided by `v4l2loopback`
```bash
ffmpeg -i /dev/video0 -f v4l2 -pix_fmt yuv420p /dev/video1
```

`ffmpeg`, if compiled with `libcaca`, is able to provide colored ASCII art output.
```bash
ffmpeg -i /dev/video0 -c:v rawvideo  -pix_fmt rgb24 -f caca -
```

## GStreamer

Another way to feed a _dummy_ device is to use `GStreamer` _pipeline_. `v4l2sink` _sink_ can be used to display video to v4l2 devices.
```
gst-launch-1.0 -v videotestsrc pattern=ball ! v4l2sink device=/dev/video1
```

# Let's go
Now lets connect everything:
  - Create a _dummy_ video device (if not already available)
```bash
sudo modprobe v4l2loopback card_label="VirtualCam #0"
```
  - Output converted video stream to ASCII art:
```bash
ffmpeg -i /dev/video0 -c:v rawvideo -pix_fmt rgb24 -f caca -
```
  - Pipeline X11 window output
```bash
gst-launch-1.0 ximagesrc xname="pipe:" ! video/x-raw,framerate=5/1 ! videoconvert ! videoscale ! "video/x-raw,format=YUY2,width=320,height=240" ! v4l2sink device=/dev/video1
```
  - Open `Zoom`. You are now able to select your _dummy_ video device you can select: _Video icon_ > _Select a camera_ > _VirtualCam #0_ and enjoy.

A better solution, that involves coding would have been to:
  - Acquire video signal from _read_ device, via V4L2 api
  - Convert steam via `libcaca`, with eventual image tuning
  - Convert back rgb24 to YUV420
  - Output video signal to _dummy_ device, via V4L2 api

For now it's good enough to play around.
