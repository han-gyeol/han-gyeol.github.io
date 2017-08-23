---
layout: post
title: 'GSoC 2017: Phase 1 Evaluation'
comments: true
category: GSoC
tags: [Jitsi, GSoC, WebRTC]
---

In the first month of the project, I have attempted following tasks:

1. Capture the large video embedded inside the Jitsi-meet’s iframe in the main renderer BrowserWindow
2. Transmit to the micro mode’s BrowserWindow
3. Display on a HTML video element


# 1. Capture the HTML Video Element inside iframe

![source]({{ site.url }}/{{ site.baseurl }}/images/2017-06-29-GSoC-2017:Phase-1-evaluation/source.png){:.center-image}

The Jitsi-meet’s largeVideo can be extracted directly from its iframe, using

{% highlight js %}
iframe.contentWindow.document.getElementById('largeVideo');
{% endhighlight %}

I can subsequently retrieve the source MediaStream from the video’s srcObject attribute. However, if I simply display that MediaStream on the micro mode’s window, it does not react when the dominant speaker changes in the Jitsi-meet application, because the original HTML video’s srcObject attributes switches to another MediaStream.

There are two possible options to implement video transition in Micro Mode:

1. A ‘hacky’ way. Import Jitsi-meet’s APP object and listen to the dominantSpeakerChanged event. Once the speaker changes, re-extract the largeVideo from the iframe.
2. Capture the largeVideo displayed on the main BrowserWindow, convert it to MediaStream and send to the Micro Mode’s window. When the speaker changes, it automatically captures the video transition.

So far, I have attempted the second approach, but it has several problems. The existing version of HTMLMediaElement.captureStream() method does not work because the largeVideo extracted from the iframe lacks ‘currentSrc’ attribute, hence keep throwing “The media element must have a source.” error. I need to find an alternative of HTMLMediaElement.captureStream() that captures a HTML video element real time, and produces a MediaStream object.

One approach I tried was using the HTML canvas to take a snapshot of each frame of the largeVideo and render it like a video.


# 2. Transmit the Video to Micro Window using WebRTC

The inherited difficulty of this task is that each Electron BrowserWindow is an independent Chromium page. There is virtually no direct way to transfer media data from one window to another. After a long research, it was concluded that using webkitRTCPeerConnection to set up a MediaStream peer connection between the main window and the micro window is the most feasible approach.

Reference: [https://www.tutorialspoint.com/webrtc/webrtc_video_demo.htm](https://www.tutorialspoint.com/webrtc/webrtc_video_demo.htm)

The details of how peer connection is implemented are shown in the [GSoC 2017: log #1](https://han-gyeol.github.io/blog//2017/05/17/GSoC-2017-Log-1/).

Since the main audio is played in background after the Jitsi-meet window is minimized, there is no need to transmit the audio to the micro window.

One major concern is the performance issue. Running a background peer connection between the main window and the micro window might cause a performance drop of the Jitsi-meet conference.


# 3. Retrieve the Video and Display on Micro Window

After the MediaStream of the largeVideo is received by the micro window’s side, it can be simply displayed by setting the srcObject attribute of the HTML video. Then, the micro mode’s window is positioned on the top right corner of the screen, with the frameless option and always-on-top options enabled. The end product looks like this…

![evaluation]({{ site.url }}/{{ site.baseurl }}/images/2017-06-29-GSoC-2017:Phase-1-evaluation/evaluation.png)

However, several problems emerged after testing minimization of the main window.

Using a HTML canvas to capture the large video works only when both windows(main and micro) are active and visible. When I tried minimizing the main window, the video in micro window just freezes. I am guessing that HTML canvas’s snapshot method works only when the target video is active(not minimized). I need to search if it is possible to play the target video in background even after the window is minimized.

Furthermore, when I tested sending a local video(mp4) from the main window to the micro window, videos in both windows ran smoothly. However, once I minimize the main window, frame rate of the micro window’s video immediately starts dropping. It is still playing, but the frame rate drops quite seriously that it does not look like a video anymore. I am going to research how Chromium browser handles a background Media, and if there is a way to activate a target MediaElement in background.


# Conclusion

Although the video transmission appears to be working, there are numerous internal problems I have to resolve before I move on to the next step. A lot of time is consumed for researching possible methods to implement the features, and only a fraction of time is used for the actual code writing. I used to only look up StackOverFlow for my programming problems, but for this project I had to read on many official API documentations, issue & bug trackers, and discussion threads. This is because the problems I used to solve were a kind that had one simple solution which worked cleanly. But for the Jitsi-meet-electron project, there are many different approaches I can take to solve the same problem, and in worst case scenario, the problem is actually unsolvable / not supported.
