---
layout: post
title: "WGL_NV_DX_interop2"
category: prog
tags: [ opengl, dxva ]
keywords: opengl, dxva, interop2
---

Name

    NV_DX_interop2

Name Strings

    WGL_NV_DX_interop2

Contributors

    Nuno Subtil, NVIDIA
    Kedarnath Thangudu, NVIDIA

Contact

    Nuno Subtil, NVIDIA Corporation (nsubtil 'at' nvidia.com)

Status

    Complete. Shipping with NVIDIA release 275 drivers, June 2011.

Version

    Last Modified Date:   October 4, 2011
    Author Revision:      2

Number

    412

Dependencies

    OpenGL 2.1 is required.
    WGL_NV_DX_interop is required.
    Windows Display Driver Model (WDDM) capable operating system 
      (such as Vista or Windows 7) required.
    Direct3D 10-capable GPU required for ID3D10Device usage.
    Direct3D 11-capable GPU required for ID3D11Device usage.

Overview

    This extension expands on the specification of WGL_NV_DX_interop
    to add support for DirectX version 10, 10.1 and 11 resources.

New Procedures and Functions

    None

New Tokens

    None

Additions to the WGL Specification

    None

Additions to the WGL_NV_DX_interop specification

    Add to the description of wglDXSetResourceShareHandleNV:

    wglDXSetResourceShareHandle does not need to be called for DirectX
    version 10 and 11 resources. Calling this function for DirectX 10
    and 11 resources is not an error but has no effect.

    Modify the last paragraph in the description of
    wglDXRegisterObjectNV:
    
    If the application explicitly requests a share handle for a DirectX
    resource, results are undefined (and may include data corruption,
    incorrect DirectX operation or program termination) if
    wglDXRegisterObjectNV is called before calling
    wglDXSetResourceShareHandleNV for the same resource. This
    restriction does not apply to non-WDDM operating systems. This
    restriction also does not apply to DirectX version 10 and 11
    resources.
    
    Add the following supported devices to table wgl.devicetypes:

    -------------------------------------------------------------------------
    DirectX device type      Device Restrictions
    -------------------------------------------------------------------------
    ID3D10Device             can only be used on WDDM operating systems;
                             Must be multithreaded
    ID3D11Device             can only be used on WDDM operating systems;
                             XXX Must be multithreaded
    -------------------------------------------------------------------------
    Table wgl.devicetypes - Valid device types for the <dxDevice> parameter of
    wglDXOpenDeviceNV and associated restrictions.
    -------------------------------------------------------------------------

    Add the following resources to table wgl.objtypes:

    --------------------------------------------------------------------------
    <type>              type of <name>     Valid DirectX resource types
    --------------------------------------------------------------------------
    TEXTURE_1D          texture            ID3D10Texture1D
                                           ID3D11Texture1D
    TEXTURE_1D_ARRAY    texture            ID3D10Texture1D
                                           ID3D11Texture1D
    TEXTURE_2D          texture            ID3D10Texture2D
                                           ID3D11Texture2D
    TEXTURE_2D_ARRAY    texture            ID3D10Texture2D
                                           ID3D11Texture2D
    TEXTURE_3D          texture            ID3D10Texture3D
                                           ID3D11Texture3D
    TEXTURE_CUBE_MAP    texture            ID3D10Texture2D
                                           ID3D11Texture2D
    TEXTURE_RECTANGLE   texture            ID3D10Texture2D
                                           ID3D11Texture2D
    RENDERBUFFER        renderbuffer       ID3D10Texture2D
                                           ID3D11Texture2D
    NONE                buffer             ID3D10Buffer
                                           ID3D11Buffer
    --------------------------------------------------------------------------
    Table wgl.objtypes - Valid values for the <type> parameter of
    wglDXRegisterObjectNV, and associated object types for GL and
    DirectX.
    --------------------------------------------------------------------------

    Add the following resources to table wgl.restrictions:

    --------------------------------------------------------------------------
    Resource Type            Resource Restrictions
    --------------------------------------------------------------------------
    ID3D10Texture1D          Usage flags must be D3D10_USAGE_DEFAULT
    ID3D10Texture2D          Usage flags must be D3D10_USAGE_DEFAULT
    ID3D10Texture3D          Usage flags must be D3D10_USAGE_DEFAULT
    ID3D10Buffer             Usage flags must be D3D10_USAGE_DEFAULT
    ID3D11Texture1D          Usage flags must be D3D11_USAGE_DEFAULT
    ID3D11Texture2D          Usage flags must be D3D11_USAGE_DEFAULT
    ID3D11Texture3D          Usage flags must be D3D11_USAGE_DEFAULT
    ID3D11Buffer             Usage flags must be D3D11_USAGE_DEFAULT
    --------------------------------------------------------------------------
    Table wgl.restrictions - Restrictions on DirectX resources that can
    be registered via wglDXRegisterObjectNV
    --------------------------------------------------------------------------

Issues

    1) How do we share D3D "tbuffers" with OpenGL?
    
    RESOLUTION: D3D "tbuffers" are analogous to buffer textures in OpenGL
    and do not have their own resource type. The ID3D1xBuffer backing the
    "tbuffer" could be shared with OpenGL and attached to a buffer texture
    by passing the interop buffer ID to TexBuffer().


Sample Code
    Example: Render to Direct3D 11 backbuffer with openGL.

    // create D3D11 device, context and swap chain.
    ID3D11Device *device;
    ID3D11DeviceContext *devCtx;
    IDXGISwapChain *swapChain;
    
    DXGI_SWAP_CHAIN_DESC scd;
    
    <set appropriate swap chain parameters in scd>
    
    hr = D3D11CreateDeviceAndSwapChain(NULL,                        // pAdapter
                                       D3D_DRIVER_TYPE_HARDWARE,    // DriverType
                                       NULL,                        // Software
                                       0,                           // Flags (Do not set D3D11_CREATE_DEVICE_SINGLETHREADED)
                                       NULL,                        // pFeatureLevels
                                       0,                           // FeatureLevels
                                       D3D11_SDK_VERSION,           // SDKVersion
                                       &scd,                        // pSwapChainDesc
                                       &swapChain,                  // ppSwapChain
                                       &device,                     // ppDevice
                                       NULL,                        // pFeatureLevel
                                       &devCtx);                    // ppImmediateContext

    // Fetch the swapchain backbuffer
    ID3D11Texture2D *dxColorbuffer;
    swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (LPVOID *)&dxColorbuffer);
    
    // Create depth stencil texture
    ID3D11Texture2D *dxDepthBuffer;
    D3D11_TEXTURE2D_DESC depthDesc;
    depthDesc.Usage = D3D11_USAGE_DEFAULT;
    <set other depthDesc parameters appropriately>
    
    // Create Views
    ID3D11RenderTargetView *colorBufferView;
    D3D11_RENDER_TARGET_VIEW_DESC rtd;
    <set rtd parameters appropriately>
    device->CreateRenderTargetView(dxColorbuffer, &rtd, &colorBufferView);
    
    ID3D11DepthStencilView *depthBufferView;
    D3D11_DEPTH_STENCIL_VIEW_DESC dsd;
    <set dsd parameters appropriately>
    device->CreateDepthStencilView(dxDepthBuffer, &dsd, &depthBufferView);
    
    // Attach back buffer and depth texture to redertarget for the device.
    devCtx->OMSetRenderTargets(1, &colorBufferView, depthBufferView);
    
    // Register D3D11 device with GL
    HANDLE gl_handleD3D;
    gl_handleD3D = wglDXOpenDeviceNV(device);

    // register the Direct3D color and depth/stencil buffers as
    // renderbuffers in opengl
    GLuint gl_names[2];
    HANDLE gl_handles[2];

    glGenRenderbuffers(2, gl_names);

    gl_handles[0] = wglDXRegisterObjectNV(gl_handleD3D, dxColorBuffer,
                                          gl_names[0],
                                          GL_RENDERBUFFER,
                                          WGL_ACCESS_READ_WRITE_NV);

    gl_handles[1] = wglDXRegisterObjectNV(gl_handleD3D, dxDepthBuffer,
                                          gl_names[1],
                                          GL_RENDERBUFFER,
                                          WGL_ACCESS_READ_WRITE_NV);

    // attach the Direct3D buffers to an FBO
    glBindFramebuffer(GL_FRAMEBUFFER, fbo);
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,
                              GL_RENDERBUFFER, gl_names[0]);
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT,
                              GL_RENDERBUFFER, gl_names[1]);
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_STENCIL_ATTACHMENT,
                              GL_RENDERBUFFER, gl_names[1]);

    while (!done) {
          <direct3d renders to the render targets>

          // lock the render targets for GL access
          wglDXLockObjectsNVX(handleD3D, 2, gl_handles);

          <opengl renders to the render targets>

          // unlock the render targets
          wglDXUnlockObjectsNVX(handleD3D, 2, gl_handles);

          <direct3d renders to the render targets and presents
           the results on the screen>
    }

Revision History

    Revision 1, 2009/06/13
     - Initial revision
    Revision 2, 2011/10/04
     - Updated supported DX10/11 resources types and their restrictions.
     - Updated the dependecies section.
     - Added an issue explaining DX "tbuffer" sharing.
     - Added sample code.