---
layout: post
title: "Sharing resources between DirectX and OpenGL"
category: prog
tags: [ opengl, directx ]
keywords: opengl, directx
---

출처:halogenica.net/sharing-resources-between-directx-and-opengl

![image](http://halogenica.net/wp-content/uploads/2014/03/Shared_Resources1.png)

I’ve recently had a need to simultaneously render using both DirectX and OpenGL. There are a number of reasons why this is useful, particularly in tools where you may wish to compare multiple rendering engines simultaneously. With this technique it is also possible to efficiently perform some rendering operations on one API to a render target, and switch to the other API to continue rendering to that render target. It can also be used to perform all rendering in a specific API, while presenting that final render target using another API. Providing direct access to textures and render targets in graphics memory regardless of API has the potential of efficiently pipelining surfaces through multiple discrete renderers.

https://github.com/halogenica/WGL_NV_DX



Generally speaking it would be advantageous to share resources (particularly surface data) between both applications. This can be achieved many ways with varying degrees of performance. However the ideal scenario would be to load only a single copy of the texture data into graphics memory, and sample from that same texture data using both DX and OGL calls at the same time. Fortunately there is an OpenGL extension to do exactly this, called WGL_NV_DX_interop for DX9, and WGL_NV_DX_interop2 for DX10+. 

This extension has historically had spotty hardware vendor support. It was initially proposed by Nvidia and support was later added to AMD graphics drivers. With the latest drivers for Intel’s current chips (HD Graphics 4200+), compatibility for this extension has been added. I’ve tested this code on an Intel HD 4400, AMD 6950, and Nvidia GTX 570. One major caveat is that I have not been able to successfully present a shared render target that is simultaneously being sampled, even with the synchronization objects available in the extension. The render target will be updated, and the sample will happen correctly, but some unknown limitation prevents the original render target from being presented. The code samples below perform a GPU copy from video memory to video memory in order to present both the DX and GL rendered results. This GPU copy is significantly faster than any CPU copy (e.g. memcpy), and is only possible through this sharing extension. However, an ideal implementation should be able to avoid this copy.

Let’s look at the special considerations for initializing DX and OGL. This demo uses DX9, however DX10+ support should also be available (I have not personally tested this). For DX9, it is important that a D3D9EX device is used (instead of a D3D9 device). D3D9EX is available in Windows 7+ and adds a new more efficient flip mode; but more importantly it enables the creation of shared resources. These resources are indicated by the handles returned by CreateOffscreenPlainSurface or CreateTexture. In the code below, CreateOffscreenPlainSurface in InitDX() will generate a shared handle, that is later registered in InitGL() using the extension function wglDXSetResourceShareHandleNV. This is what makes the GL renderer aware of the DX resource. This handle represents the GPU video memory that backs the respective DX/GL textures. In this demo, the DX renderer’s shared texture is called g_pSharedSurface and it’s handle is called g_hSharedSurface, while the GL renderer’s texture is called g_GLTexture and the corresponding handle is called g_hGLSharedTexture. Before registering the shared resource with GL, the DX device must be associated using wglDXOpenDeviceNV.



```
void InitDX(HWND hWndDX)
{
     D3DPRESENT_PARAMETERS d3dpp;

     ZeroMemory(&d3dpp, sizeof(d3dpp));
     d3dpp.Windowed = TRUE;
     d3dpp.SwapEffect = D3DSWAPEFFECT_DISCARD;
     d3dpp.hDeviceWindow = hWndDX;
     d3dpp.BackBufferFormat = D3DFMT_X8R8G8B8;

     HRESULT hr = S_OK;

     // A D3D9EX device is required to create the g_hSharedSurface 
     Direct3DCreate9Ex(D3D_SDK_VERSION, &g_pD3d);

     // The interop definition states D3DCREATE_MULTITHREADED is required, but it may vary by vendor
     hr = g_pD3d->CreateDeviceEx(D3DADAPTER_DEFAULT,
                       D3DDEVTYPE_HAL,
                       hWndDX,
                       D3DCREATE_HARDWARE_VERTEXPROCESSING | D3DCREATE_MULTITHREADED,
                       &d3dpp,
                       NULL,
                       &g_pDevice);

     hr = g_pDevice->GetRenderTarget(0, &g_pSurfaceRenderTarget);
     D3DSURFACE_DESC rtDesc;
     g_pSurfaceRenderTarget->GetDesc(&rtDesc);

     // g_pSharedSurface should be able to be opened in OGL via the WGL_NV_DX_interop extension
     // Vendor support for various textures/surfaces may vary
     hr = g_pDevice->CreateOffscreenPlainSurface(rtDesc.Width, 
                                                 rtDesc.Height, 
                                                 rtDesc.Format, 
                                                 D3DPOOL_DEFAULT, 
                                                 &g_pSharedSurface, 
                                                 &g_hSharedSurface);

     hr = g_pDevice->SetRenderState(D3DRS_LIGHTING, FALSE);
     hr = g_pDevice->SetRenderState(D3DRS_CULLMODE, D3DCULL_NONE);


     CUSTOMVERTEX vertices[] = 
     {
         {  2.0f, -2.0f, 0.0f, D3DCOLOR_XRGB(0, 0, 255)},
         {  0.0f,  2.0f, 0.0f, D3DCOLOR_XRGB(0, 255, 0)},
         { -2.0f, -2.0f, 0.0f, D3DCOLOR_XRGB(255, 0, 0)},
     };

     hr = g_pDevice->CreateVertexBuffer(9*sizeof(CUSTOMVERTEX),
                                        0,
                                        CUSTOMFVF,
                                        D3DPOOL_DEFAULT,
                                        &g_pVB,
                                        NULL);

     VOID* pVoid;
     hr = g_pVB->Lock(0, 0, (void**)&pVoid, 0);
     memcpy(pVoid, vertices, sizeof(vertices));
     hr = g_pVB->Unlock();
}


void InitGL(HWND hWndGL)
{
     static  PIXELFORMATDESCRIPTOR pfd=
     {
         sizeof(PIXELFORMATDESCRIPTOR),              // Size Of This Pixel Format Descriptor
         1,                                          // Version Number
         PFD_DRAW_TO_WINDOW |                        // Format Must Support Window
         PFD_SUPPORT_OPENGL |                        // Format Must Support OpenGL
         PFD_DOUBLEBUFFER,                           // Must Support Double Buffering
         PFD_TYPE_RGBA,                              // Request An RGBA Format
         32,                                         // Select Our Color Depth
         0, 0, 0, 0, 0, 0,                           // Color Bits Ignored
         0,                                          // No Alpha Buffer
         0,                                          // Shift Bit Ignored
         0,                                          // No Accumulation Buffer
         0, 0, 0, 0,                                 // Accumulation Bits Ignored
         16,                                         // 16Bit Z-Buffer (Depth Buffer)  
         0,                                          // No Stencil Buffer
         0,                                          // No Auxiliary Buffer
         PFD_MAIN_PLANE,                             // Main Drawing Layer
         0,                                          // Reserved
         0, 0, 0                                     // Layer Masks Ignored
     };
     
     g_hDCGL = GetDC(hWndGL);
     GLuint PixelFormat = ChoosePixelFormat(g_hDCGL, &pfd);
     SetPixelFormat(g_hDCGL, PixelFormat, &pfd);
     HGLRC hRC = wglCreateContext(g_hDCGL);
     wglMakeCurrent(g_hDCGL, hRC);

     GLenum x = glewInit();

     // Register the shared DX texture with OGL
     if (WGLEW_NV_DX_interop)
     {
         // Acquire a handle to the D3D device for use in OGL
         g_hDX9Device = wglDXOpenDeviceNV(g_pDevice);

         if (g_hDX9Device)
         {
             glGenTextures(1, &g_GLTexture);

             // This registers a resource that was created as shared in DX with its shared handle
             bool success = wglDXSetResourceShareHandleNV(g_pSharedSurface, g_hSharedSurface);

             // g_hGLSharedTexture is the shared texture data, now identified by the g_GLTexture name
             g_hGLSharedTexture = wglDXRegisterObjectNV(g_hDX9Device,
                                                        g_pSharedSurface,
                                                        g_GLTexture,
                                                        GL_TEXTURE_2D,
                                                        WGL_ACCESS_READ_ONLY_NV);
         }
     }

     glViewport(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT);
     glMatrixMode(GL_PROJECTION);
     glLoadIdentity();
     glOrtho(0, SCREEN_WIDTH, SCREEN_HEIGHT, 0, -1, 1);
     glDisable(GL_DEPTH_TEST);
     glMatrixMode(GL_MODELVIEW);
     glLoadIdentity();
}
```


After the textures are created as shared surfaces and these surfaces are registered with the GL renderer, we can begin rendering. The most important consideration here is proper synchronization of reads/writes of the shared surface between the two renderers. This example shows the DX renderer writing a triangle to a render target, which is then copied to the shared surface, and the GL renderer reading from that shared surface and texturing a quad with it. Because of the potential read-after-write hazards, it is necessary for the GL renderer to acquire a lock on the shared surface via wglDXLockObjectsNV. This lock does not result in the surface being copied to CPU space (as would happen with a normal DX9 Lock operation. Instead, this lock triggers the GPU to perform the necessary flushing and stalling to guarantee that the surface has finished being written to before reading from it. This is necessary even if the DX and GL renderers operate sequentially (i.e. not multi-threaded), because the rendering commands scheduled on the GPU execute asynchronously.



```
void RenderDX(void)
{    
     // Set up transformations
     D3DXMATRIX matView;
     D3DXMatrixLookAtLH(&matView,
                        &D3DXVECTOR3 (0.0f, 0.0f, -10.0f),
                        &D3DXVECTOR3 (0.0f, 0.0f, 0.0f),
                        &D3DXVECTOR3 (0.0f, 1.0f, 0.0f));
     g_pDevice->SetTransform(D3DTS_VIEW, &matView);

     D3DXMATRIX matProjection;
     D3DXMatrixPerspectiveFovLH(&matProjection,
                                D3DXToRadian(45),
                                (FLOAT)SCREEN_WIDTH / (FLOAT)SCREEN_HEIGHT,
                                1.0f,
                                25.0f);
     g_pDevice->SetTransform(D3DTS_PROJECTION, &matProjection);

     D3DXMATRIX matTranslate;
     D3DXMatrixTranslation(&matTranslate, 0.0f, 0.0f, 0.0f);

     D3DXMATRIX matRotate;
     static float rot = 0; 
     rot+=0.01;
     D3DXMatrixRotationZ(&matRotate, rot);

     D3DXMATRIX matTransform = matRotate * matTranslate;

     HRESULT hr = S_OK;
     hr = g_pDevice->Clear(0, NULL, D3DCLEAR_TARGET, D3DCOLOR_XRGB(40, 40, 60), 1.0f, 0);

     // Draw a spinning triangle
     hr = g_pDevice->BeginScene();

     hr = g_pDevice->SetStreamSource(0, g_pVB, 0, sizeof(CUSTOMVERTEX));
     hr = g_pDevice->SetFVF(CUSTOMFVF);
     hr = g_pDevice->SetTransform(D3DTS_WORLD, &matTransform);
     hr = g_pDevice->DrawPrimitive(D3DPT_TRIANGLELIST, 0, 1);

     // Copy the render target to the shared surface
     // StretchRect between two D3DPOOL_DEFAULT surfaces will be a GPU Blt.
     // Note that GetRenderTargetData() cannot be used because it is intended to copy from GPU to CPU.
     hr = g_pDevice->StretchRect(g_pSurfaceRenderTarget, NULL, g_pSharedSurface, NULL, D3DTEXF_NONE);
     hr = g_pDevice->EndScene();
     hr = g_pDevice->Present(NULL, NULL, NULL, NULL);
}

void RenderGL(void)
{
     glClearColor(0.0, 0.0, 0.0, 1.0);
     glClear(GL_COLOR_BUFFER_BIT);

     // Lock the shared surface
     wglDXLockObjectsNV(g_hDX9Device, 1, &g_hGLSharedTexture);

     glEnable(GL_TEXTURE_2D);
     glBindTexture(GL_TEXTURE_2D, g_GLTexture);

     glPushMatrix();
     glBegin(GL_QUADS);

     glTexCoord2d(0.0, 0.0); glVertex2f(         0.0f,           0.f);
     glTexCoord2d(0.0, 1.0); glVertex2f(         0.0f, SCREEN_HEIGHT);
     glTexCoord2d(1.0, 1.0); glVertex2f( SCREEN_WIDTH, SCREEN_HEIGHT);
     glTexCoord2d(1.0, 0.0); glVertex2f( SCREEN_WIDTH,          0.0f);

     glEnd();
     glPopMatrix();

     SwapBuffers(g_hDCGL);

     // Unlock the shared surface
     wglDXUnlockObjectsNV(g_hDX9Device, 1, &g_hGLSharedTexture);
}
```

The final tear down step is extremely straight forward. The call to wglDXUnregisterObjectNV will disassociate the shared resource with GL, and wglDXCloseDeviceNV will close the device that created the shared surface.




```
void Destroy(void)
{
     if (WGLEW_NV_DX_interop)
     {
         if (g_hGLSharedTexture)
             wglDXUnregisterObjectNV(g_hDX9Device, g_hGLSharedTexture);
         if (g_hDX9Device)
             wglDXCloseDeviceNV(g_hDX9Device);
     }

     if (g_pSysmemSurface)
         g_pSysmemSurface->Release();
     if (g_pSharedSurface)
         g_pSharedSurface->Release();
     if (g_pSharedTexture)
         g_pSharedTexture->Release();
     if (g_pSurfaceRenderTarget)
         g_pSurfaceRenderTarget->Release();
     if (g_pVB)
         g_pVB->Release();
     if (g_pDevice)
         g_pDevice->Release();
     if (g_pD3d)
         g_pD3d->Release();
}
```