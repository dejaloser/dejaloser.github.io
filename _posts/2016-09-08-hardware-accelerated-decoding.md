---
layout: post
title: "Hardware Accelerated Decoding"
category: prog
tags: decoding
keywords: decoding
---

출처: https://github.com/wang-bin/QtAV/wiki/Hardware-Accelerated-Decoding

VA-API

Install vainfo libva-dev, libva-intel-vaapi-driver(intel gpu), xvba-va-driver(ATI), vdpau-va-driver(NVIDIA)

If vainfo works, then QtAV should work with VA-API

EGL

Since Qt5.5, you can enable EGL by setting environment var  QT_XCB_GL_INTEGRATION=xcb_egl .  player  and  QMLPlayer  have EGL configuration too. If EGL is used, VA-API decoding and OpenGL rendering can get the best performance. NVIDIA GPUs currently do not support EGL+VA-API. AMD is not tested.

If EGL is used, you can use zero copy for X11, DRM display. Otherwise, zero copy is only available for X11 and GLX display.

CUDA

OpenGL desktop mode supports 0-copy. To use 0-copy, make sure your program runs with NVIDIA GPU (on Optimus).  DirectCopy  is the default.  DirectCopy  mode is fast enough, faster than potplayer, lavfilters+mpc, and may be the fastest one.  ZeroCopy  mode is even faster.

Linux

On ubuntu 16.04, nvidia-361-updates should work.

Windows

Install latest nvidia driver.

DXVA/D3D11

D3D11 decoder is supported for windows store. Run in OpenGL ES2 mode (ANGLE) to enable 0-copy. DXVA 0-copy in OpenGL desktop mode only supports some new intel GPUs in my tests, maybe also supports AMD.

Developer

Build

Code

CUDA:
QVaraiantHash cuda_opt;
cuda_opt["surfaces"] = 0; //key is property name, case sensitive
cuda_opt["copyMode"] = "ZeroCopy"; // default is "DirectCopy"
QVariantHash opt;
opt["CUDA"] = cuda_opt; //key is decoder name, case sensitive
player.setOptionsForVideoCodec(opt);


Since QtAV1.8, CUDA decoder supports ZeroCopy. Run your program with NVIDIA GPU if you have multiple GPUs.

VA-API:
QVaraiantHash va_opt;
va_opt["display"] = "X11"; //"GLX", "X11", "DRM"
va_opt["copyMode"] = "ZeroCopy"; // "ZeroCopy", "OptimizedCopy", "GenericCopy". Default is "ZeroCopy" if possible
QVariantHash opt;
opt["VAAPI"] = va_opt; //key is decoder name, case sensitive
player.setOptionsForVideoCodec(opt);


Copy Mode

Copy mode for GPU decoders was introduced since 1.5.0. Copy mode can be  ZeroCopy ,  LazyCopy ,  OptimizedCopy  and  GenericCopy .
•ZeroCopy: No data read back from GPU. Directly render the decoded data on GPU. It's the most efficient way. Now it's supported by ◦CUDA
◦VA-API with libva-x11 and libva-glx. It may not work for vdpau backend.
◦VDA(OSX), VideoToolbox(OSX, iOS>=8)
◦DXVA with ANGLE. (Desktop OpenGL is not stable now)

•LazyCopy: Used in VDA and VideoToolBox
•OptimizedCopy: Use SSE4.1 for x86 used by DXVA and VA-API(VDA crashes), and NEON for CedarV. SSE4.1 does not work for some GPUs, e.g. NVIDIA GS8400
•GenericCopy: Use memcpy

Status
•D3D11  Copy
 EGL RGB (D3D11)
 EGL YUV

•DXVA  Copy
 EGL (D3D9)
 EGL DXVAHD
 OpenGL

•CUDA  Copy
 EGL (WIP)
 OpenGL

•VA-API  Copy
 EGL DMA/DRM
 EGL TFP
 GLX TFP

•VDA  Copy
 OpenGL

•VideoToolbox  OSX Copy/0-Copy
 iOS Copy/0-Copy

 VDPAU
 OMX
