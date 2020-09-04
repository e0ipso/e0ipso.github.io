---
title: 'Guvcview instead of Cheese'
categories:
  - linux
tags:
  - Free Software
  - linux
  - parenting
image: /assets/images/2020/webcam.jpg
---
I have been experiencing [significant slowness](https://youtube.com/watch?v=JgwnhG2Nx74) in my webcam using cheese. I
have moved to Guvcview now, and I am very happy with the move.

Almost a year ago I started a private video blog project. I sit in front of the camera and address to my kids as if they
were watching this in the future. In these videos I tell them all the fun things we have done throughout the previous
day. My goal is for the three of us to have a record of how happy their toddler years were.

<!-- more -->

Just like with my OBS recordings, the video was very laggy, evey with all the might of my desktop computer. The effect
was as if I was recording the video in slow motion. This has bothered me enough to look into what is going on. My
investigation led me to learn that if I use [YUYV](https://en.wikipedia.org/wiki/YUV) as the output format for my
webcam, the image becomes slow. As if there was a ghost in front of the computer instead of me. On the other side, if I
use [MJPEG](https://en.wikipedia.org/wiki/Motion_JPEG) instead things look great.

This was only happening with some programs. I suspect that this is because the defaults for my webcam's driver use MJPEG
already. To learn this I installed (in Ubuntu) _Video for Linux Utils_ temporarily:

```
sudo apt install v4l-utils
```

With that I could learn more about the default parameters my webcam was operating under. I used this command for that:

```
v4l2-ctl --all
```

Among other parameters I could see the following:

```
Video input : 0 (Camera 1: ok)
Format Video Capture:
        Width/Height      : 1920/1080
        Pixel Format      : 'MJPG' (Motion-JPEG)
```

That was when I realized that there was nothing wrong with my camera, but with the software I was using it with. With
OBS it was very simple to fix. I configured it to use some other output format instead of YUYV. However, I had no
control on that for Cheese, the program I use for my video blog. I was sad to learn that because I really like Cheese's
UX, it reminds me of Photo Booth in my old mac days.

![Screenshot Cheese](/assets/images/cheese.png)

After some research and testing I arrived at the conclusion that the best match for me is Guvcview
(`sudo apt install guvcview`). It does not have that same polished edge, but it is a fantastic piece of software! It has
all the control I needed in Cheese and more. The only thing I could not get exactly as I wanted was that I wanted
Guvcview to create each new recording with the date in the filename. I could not find a way to make it do that. I was
after `$(date +'%Y-%m-%d').mkv`. That's not a big deal, I can rename the video blog entry manually.

![Screenshot Guvcview](/assets/images/guvcview.png)

---
<small>Photo by [kevin Ruiz chávez](https://unsplash.com/@kevin_2?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/webcam?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) </small>
