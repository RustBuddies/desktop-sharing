# desktop-sharing

Building a remote desktop sharing application using webRTC

## Overview

We'll be building a remote desktop sharing application i.e. a TeamViewer clone using webRTC. Efforts on this front have already been made - [Rust Desk](https://github.com/rustdesk/rustdesk) being a great example. The main motivation is to learn Rust by building a real-world project.

## Technical Draft

This draft is going to serve as a reference for all technical specifications, requirements and high-level documentation for the project. It is bound to change/update as the project takes shape and also subject to any future requirements/objective changes.

### Functional Requirement

The application must allow two peers to share their screen with each other, where one peer (remote) can take control of the mouse/keyboard of the other peer (host)

### Technical Requirements

To build such an application we would need to implement or use crates for the following mechanisms:

- Capturing screen
- Encoding captured screen data with either h264, vpx codec
- Streaming the encoded video data to the remote peer
- Decoding the host peer video stream
- Displaying the host peer's screen in an application window
- Capturing the remote peer's mouse/keyboard inputs from the application window
- Transmitting said data to host peer
- Enacting/translating/mimicking the mouse/keyboard events from the remote peer on the host's machine

#### What's already established

##### Screen Capture and Video Encoding/Decoding

Screen capture is to be done using [gstreamer](https://gstreamer.freedesktop.org/). However, gstreamer is a really complex framework and learning it in and of itself can take a lot of effort and time. We would be better off finding an alternative. One of the alternative is to take a look at how rustdesk does it.

##### Streaming

We will be using webRTC for streaming video and mouse/keyboard data. [webRTC for the curious](https://webrtcforthecurious.com/) is an awesome resource for learning ins and outs of webRTC.

The 'track' api should allow us to stream video and we'll use 'data channel' for transmitting the keyboard/mouse events.

It is also absolutely necessary to build a signalling server that's going to facilitate setting up the call between the peers. A webRTC connection can not be made without the signalling server. How the signalling server is implemented is totally up for discussion.

##### Display Window/ GUI

For this part we will have to use a GUI framework. It can take a lot of figuring out how to actually display a stream of data (live-video) on a UI canvas/widget and keeping it from lagging behind. An immediate-mode GUI framework will be the best way to do this as they tend to offer per-frame rendering control in an event loop.

The same GUI framework should also allow capturing mouse/keyboard interactions.

#### Tying it all together

The application will be operating in one of two modes:

1. Host Mode

- The application will capture the host screen and encode it.
- As soon as a call request comes in from the signalling server, the host will answer it to negotiate following the webRTC offer/answer/ICE flow
- On a successful negotiation, the host will relay its video track through the
peer connection
- The host will be listening for data on its end of the data channel
- On receiving data on the data channel, the host will have to translate the mouse/keyboard events

2. Remote Mode

- As remote, the application is able to make a call request to another peer by talking to the signalling server
- After successful negotiation with the host peer, the application will display a wundow in which the host's video stream will be displayed.
- The remote peer's mouse/keyboard inputs will be captured by the same window when in focus so as to avoid sending input events if the window is in background/ not in focus.