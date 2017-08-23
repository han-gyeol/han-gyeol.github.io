---
layout: post
title: 'GSoC 2017: Phase 2 Evaluation'
comments: true
category: GSoC
tags: [Jitsi, GSoC, WebRTC]
---

By the end of Phase 2, I have completed following tasks:

1. Create Micro window and transfer MediaElement to it.
2. Implement video transition mechanism reacting on speaker change
3. Develop basic toolbar buttons embedded in micro mode
4. Modularize and refactor the micro mode code

[https://github.com/jitsi/jitsi-meet-electron/pull/12/commits](https://github.com/jitsi/jitsi-meet-electron/pull/12/commits)

There are two modules I created for this project: ‘p2pconnection’ and ‘micromode’.

‘p2pconnection’ module is responsible for transferring the Jitsi-meet MediaElement from one Electron BrowserWindow to another using WebRTC technology. More details can be found in my previous post.

‘micromode’ module is simply a complication of all the codes I have written onto the existing codebase to make the Micro mode work. Instead of writing everything on the main.js and render.js files, I have refactored them out as a module for better readability. Currently, Jitsi-meet-electron app’s code consists of three main parts: main.js, render.js and micro.js files. ‘main.js’ file is the Main Process part of the Electron framework while ‘render.js’ and ‘micro.js’ are the Renderer Process part of the Electron, each responsible for running a BrowserWindow.

In each process, respective part of ‘micromode’ module is imported, initialized and disposed once the application is closed.

![code]({{ site.url }}/{{ site.baseurl }}/images/2017-07-28-GSoC-2017:Phase-2-evaluation/code.png){:.center-image}

As shown above, the main process simply has to require the ‘micromode’ module, and call inti, show, hide, dispose methods whenever they are necessary.



There are a few potential areas of improvement from the current version:


## 1. Add more features to Micro mode’s toolbar

Micro mode currently has audio mute, video mute and hangup features, but there are plenty of other functionalities in the Jitsi-meet application that Micro mode can also provide, such as chat and screen sharing. Some of the features are not suitable for Micro mode, such as live stream, shared document and shared YouTube video. Screen sharing feature especially goes well with the Micro mode because the user might want to show their desktop and be able to watch the remote video at the same time.

## 2. Optimize the video element in Micro mode.

Micro mode occasionally has a lagging issue, either the video is a fraction of a second slower than the original video, and the transition animation is sometimes clunky. This is probably because of the WebRTC video transmission overhead or too much resources taken by Micro window. One thing I can try at the moment is to lower the video resolution in the Micro mode.

## 3. Switch to more reliable WebRTC technologies.

Currently, Micro mode’s modules several WebRTC experimental functionalities which are quite unstable and some of which are deprecated. I had no choice because there is no practical alternatives. As WebRTC get more developed, there should be follow-up maintenance works to switch to newer and more stable technologies.


# Conclusion

The more I work on this project, the more I realize the lack of IPC supports provided by Electron framework which causes spaghetti codes and runtime overhead, while giving me tremendous amount of headache during development. Nonetheless, the current version of Micro mode finally functions as its purpose, so the rest of the development would be mainly optimization works and adding more functionalities to it.
