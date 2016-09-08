---
layout: post
title: "Decoding and Rendering Benchmarks"
category: prog
tags: benchmarks
keywords: decoding
---

출처:http://halogenica.net/sharing-resources-between-directx-and-opengl

Decoding and Rendering Benchmarks
Our decoding and rendering benchmarks consists of standardized test clips (varying codecs, resolutions and frame rates) being played back through MPC-HC. GPU usage is tracked through GPU-Z logs and power consumption at the wall is also reported. The former provides hints on whether frame drops could occur, while the latter is an indicator of the efficiency of the platform for the most common HTPC task - video playback.

Enhanced Video Renderer (EVR) / Enhanced Video Renderer - Custom Presenter (EVR-CP)

The Enhanced Video Renderer is the default renderer made available by Windows 8. It is a lean renderer in terms of usage of system resources since most of the aspects are offloaded to the GPU drivers directly. EVR is mostly used in conjunction with native DXVA2 decoding. The GPU is not taxed much by the EVR despite hardware decoding also taking place. Deinterlacing and other post processing aspects were left at the default settings in the Intel HD Graphics Control Panel (and these are applicable when EVR is chosen as the renderer). EVR-CP is the default renderer used by MPC-HC. It is usually used in conjunction with MPC-HC's video decoders, some of which are DXVA-enabled. However, for our tests, we used the DXVA2 mode provided by the LAV Video Decoder. In addition to DXVA2 Native, we also used the QuickSync decoder developed by Eric Gur (an Intel applications engineer) and made available to the open source community. It makes use of the specialized decoder blocks available as part of the QuickSync engine in the GPU.

![image](https://dejaloser.github.io/resource/image/55350.png)


Haswell HTPC  Ivy Bridge Passive HTPC
Power consumption shows a tremendous decrease across all streams. Admittedly, the passive Ivy Bridge HTPC uses a 55W TDP Core i3-3225, but, as we will see later, the power consumption at full load for the Haswell build is very close to that of the Core i3-3225 build despite the lower TDP of the Core i7-4765T.

In general, using the QuickSync decoder results in a higher power consumption because the decoded frames are copied back to the DRAM before being sent to the renderer. Using native DXVA decoding, the frames are directly passed to the renderer without the copy-back step. The odd-man out in the power numbers is the interlaced VC-1 clip, where QuickSync decoding is more efficient compared to 'native DXVA2'. This is because there is currently no support in the open source native DXVA2 decoders for interlaced VC-1 on Intel GPUs, and hence, it is done in software. On the other hand, the QuickSync decoder is able to handle it with the VC-1 bitstream decoder in the GPU.

![image](https://dejaloser.github.io/resource/image/55349.png)

Haswell HTPC  Ivy Bridge Passive HTPC
The GPU utilization numbers follow a similar track to the power consumption numbers. EVR is very lean on the GPU, as discussed earlier. The utilization numbers provide proof of the same. QuickSync appears to stress the GPU more, possibly because of the copy-back step for the decoded frames.

madVR

Videophiles often prefer madVR as their renderer because of the choice of scaling algorithms available as well as myriad other features. In our recent Ivy Bridge HTPC review, we found that with DDR3-1600 DRAM, it was straightforward to get madVR working with the default scaling algorithms for all materials 1080p60 or lesser. In the meanwhile, Mathias Rauen (developer of madVR) has developed more features. In order to alleviate the ringing artifacts introduced by the Lanczos algorithm, an option to enable an anti-ringing filter was introduced. A more intensive scaling algorithm (Jinc) was also added. Unfortunately, enabling either the anti-ringing filter with Lanczos or choosing any variant of Jinc resulted in a lot of dropped frames. Haswell's HD4600 is simply not powerful enough for these madVR features.

It is not possible to use native DXVA2 decoding with madVR because the decoded frames are not made available to an external renderer directly. (Update: It is possible to use DXVA2 Native with madVR since v0.85. Future HTPC articles will carry updated benchmarks) To work around this issue, LAV Video Decoder offers three options. The first option involves using software decoding. The second option is to use either QuickSync or DXVA2 Copy-Back. In either case, the decoded frames are brought back to the system memory for madVR to take over. One of the interesting features to be integrated into the recent madVR releases is the option to perform DXVA scaling. This is particularly interesting for HTPCs running Intel GPUs because the Intel HD Graphics engine uses dedicated hardware to implement support for the DXVA scaling API calls. AMD and NVIDIA apparently implement those calls using pixel shaders. In order to obtain a frame of reference, we repeated our benchmark process using DXVA2 scaling for both luma and chroma instead of the default settings.


![image](https://dejaloser.github.io/resource/image/55352.png)

Haswell HTPC  Ivy Bridge Passive HTPC
One of the interesting aspects to note here is the fact that the power consumption numbers show a much larger shift towards the lower end when using DXVA2 scaling. This points to more power efficient updates in the GPU video post processing logic.

![image](https://dejaloser.github.io/resource/image/55351.png)

Haswell HTPC  Ivy Bridge Passive HTPC
DXVA scaling results in much lower GPU usage for SD material in particular with a corresponding decrease in average power consumption too. Users with Intel GPUs can continue to enjoy other madVR features while giving up on the choice of a wide variety of scaling algorithms.