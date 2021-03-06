---
layout: post
title: "WGL_NV_DX_interop"
category: prog
tags: [ opengl, dxva ]
keywords: opengl, dxva, interop
---

Name

    NV_DX_interop

Name Strings

    WGL_NV_DX_interop

Contributors

    Michael Gold, NVIDIA
    Nuno Subtil, NVIDIA

Contact

    Nuno Subtil, NVIDIA Corporation (nsubtil 'at' nvidia.com)

Status

    Complete. Shipping with NVIDIA release 265 drivers, November 2010.

Version

    Last Modified Date:         10/11/2010
    Revision:                   1

Number

    407

Dependencies

    OpenGL 2.1 is required.

Overview

    This extension allows OpenGL to directly access DirectX buffers
    and surfaces.  A DirectX vertex buffer may be shared as an OpenGL
    buffer object and a DirectX surface may be shared as an OpenGL
    texture or renderbuffer object.

New Procedures and Functions

    BOOL wglDXSetResourceShareHandleNV(void *dxObject, HANDLE shareHandle);

    HANDLE wglDXOpenDeviceNV(void *dxDevice);
    BOOL wglDXCloseDeviceNV(HANDLE hDevice);
    HANDLE wglDXRegisterObjectNV(HANDLE hDevice, void *dxObject,
                                  GLuint name, GLenum type, GLenum access);
    BOOL wglDXUnregisterObjectNV(HANDLE hDevice, HANDLE hObject);
    BOOL wglDXObjectAccessNV(HANDLE hObject, GLenum access);
    BOOL wglDXLockObjectsNV(HANDLE hDevice, GLint count, HANDLE *hObjects);
    BOOL wglDXUnlockObjectsNV(HANDLE hDevice, GLint count, HANDLE *hObjects);

New Tokens

    Accepted by the <access> parameters of wglDXRegisterObjectNV and
    wglDXObjectAccessNV:

      WGL_ACCESS_READ_ONLY_NV             0x0000
      WGL_ACCESS_READ_WRITE_NV            0x0001
      WGL_ACCESS_WRITE_DISCARD_NV         0x0002

Additions to the WGL Specification

    OpenGL may directly access textures, surfaces and buffers created
    by DirectX.  A DirectX device is prepared for interoperability
    by calling

        HANDLE wglDXOpenDeviceNV(void *dxDevice);

    <dxDevice> is a pointer to a supported Direct3D device
    object. Supported devices are listed in the wgl.devicetypes table,
    along with applicable restrictions for each device. The return
    value is a handle to a GL/DirectX interop device.

    When wglDXOpenDeviceNV fails to open a Direct3D device, NULL is
    returned. To get extended error information, call GetLastError.
    Possible errors are as follows:

      ERROR_OPEN_FAILED        Could not open the Direct3D device.

      ERROR_NOT_SUPPORTED      The <dxDevice> is not supported.
                               This can be caused either by passing in
                               a device from an unsupported DirectX
                               version, or by passing in a device
                               referencing a display adapter that is
                               not accessible to the GL.

    Calling this entrypoint with an invalid <dxDevice> pointer results
    in undefined behavior and may result in data corruption or program
    termination.

    Versions of the operating system that support the Windows Display
    Driver Model (WDDM) support a sharing mechanism for DirectX
    resources that makes use of WDDM share handles. The application may
    obtain a share handle from the operating system to share a surface
    among several Direct3D devices.

    This extension accommodates, but does not require use of WDDM share
    handles. An application should only obtain a WDDM share handle at
    resource creation time if it will share the resource with non-GL
    clients.

    As of today, all versions of Microsoft Windows starting with Windows
    Vista use the Windows Display Driver Model, enabling the usage of
    WDDM share handles.

    If the application wishes to share a DirectX version 9 resource
    under a WDDM operating system, it is required that the Direct3D
    device that owns the resource be a Direct3D9Ex device.

    -------------------------------------------------------------------------
    DirectX device type      Device Restrictions
    -------------------------------------------------------------------------
    IDirect3DDevice9         can not be used on WDDM operating systems;
                             D3DCREATE_MULTITHREADED behavior flag must be
                             set at device creation time

    IDirect3DDevice9Ex       D3DCREATE_MULTITHREADED behavior flag must be
                             set at device creation time
    -------------------------------------------------------------------------
    Table wgl.devicetypes - Valid device types for the <dxDevice> parameter of
    wglDXOpenDeviceNV and associated restrictions.
    -------------------------------------------------------------------------

    If the application wishes to share a DirectX resource with GL and
    non-GL clients, it should request a share handle for the
    resource. It must also call

        BOOL wglDXSetResourceShareHandleNV(void *dxResource, HANDLE shareHandle);

    to associate the share handle with the DirectX resource prior to
    registering it with the GL. <dxResource> is a pointer to the DirectX
    resource that will be shared, <shareHandle> contains the share
    handle that the OS generated for the resource.

    The return value for wglDXSetResourceShareHandleNV is FALSE if
    this function is called more than once for any resource; called
    with an invalid <shareHandle> parameter; or called for a resource
    that is already being shared with the GL; otherwise, the return
    value is TRUE. On a version of the OS that does not support WDDM,
    calling wglDXSetResourceShareHandleNV returns TRUE but has no
    effect.

    Results are undefined if the <dxResource> pointer is invalid and
    may result in data corruption or program termination.

    Calling

        HANDLE wglDXRegisterObjectNV(HANDLE hDevice, void *dxResource,
                                     GLuint name, GLenum type, GLenum access);

    prepares a DirectX object for use by the GL.  <hDevice> is a
    GL/DirectX interop device handle, as returned by wglDXOpenDeviceNV.

    <dxResource> is a pointer to a DirectX resource to be registered
    with the GL. The resource must be of one of the types identified in
    table wgl.objtypes and must also obey the restrictions specified for
    that resource in table wgl.restrictions.

    <type> identifies the GL object type that will map to the DirectX
    resource being shared and must be one of the types enumerated in
    table wgl.objtypes.  <name> is the GL object name to be assigned
    to the DirectX resource in the namespace of the objects identified
    by <type> in the current GL context. The valid combinations of
    <type> and <dxResource> are described in table wgl.objtypes.

    <access> indicates the intended usage of the resource in GL and must
    be one of the following:

        WGL_ACCESS_READ_ONLY_NV indicates that GL will read the
        DirectX resource but will not modify it.  If GL attempts to
        modify the data store of the resource, the result is undefined
        and may include program termination.

        WGL_ACCESS_READ_WRITE_NV indicates that GL may read or write
        the data store of the resource.

        WGL_ACCESS_WRITE_DISCARD_NV indicates that any previous
        contents of the object may be discarded and GL may write to
        the resource.  Any data written by GL may be reliably read in
        subsequent operations.  The result of subsequently reading
        data outside the region written by GL is undefined.

    If the call is successful, the return value is a handle to a
    GL/DirectX interop object. A return value of NULL indicates that
    an error occurred. To obtain extended error information, call
    GetLastError. Possible errors are as follows:

      ERROR_INVALID_HANDLE    No GL context is made current to the
                              calling thread.

      ERROR_INVALID_DATA      Incorrect <name> <type> or <access>
                              parameters.

      ERROR_OPEN_FAILED       Opening the Direct3D resource failed.

    Calling wglDXRegisterObjectNV with an invalid <hDevice> handle
    results in undefined behavior and may result in data corruption or
    program termination.

    If the application explicitly requests a share handle for a
    DirectX resource, results are undefined (and may result in data
    corruption, incorrect DirectX operation or program termination) if
    wglDXRegisterObjectNV is called before calling
    wglDXSetResourceShareHandleNV for the same resource. This
    restriction does not apply to non-WDDM operating systems.

    --------------------------------------------------------------------------
    <type>              type of <name>     Valid DirectX resource types
    --------------------------------------------------------------------------
    TEXTURE_2D          texture            IDirect3DSurface9
                                           IDirect3DTexture9

    TEXTURE_3D          texture            IDirect3DVolumeTexture9

    TEXTURE_CUBE_MAP    texture            IDirect3DCubeTexture9

    TEXTURE_RECTANGLE   texture            IDirect3DSurface9
                                           IDirect3DTexture9

    RENDERBUFFER        renderbuffer       IDirect3DSurface9

    NONE                buffer             IDirect3DIndexBuffer9
                                           IDirect3DVertexBuffer9
    --------------------------------------------------------------------------
    Table wgl.objtypes - Valid values for the <type> parameter of
    wglDXRegisterObjectNV and associated object types for GL and
    DirectX.
    --------------------------------------------------------------------------

    --------------------------------------------------------------------------
    Resource Type            Resource Restrictions
    --------------------------------------------------------------------------
    IDirect3DSurface9        Must not be lockable

    IDirect3DTexture9        Memory pool must be D3DPOOL_DEFAULT

    IDirect3DCubeTexture9    Memory pool must be D3DPOOL_DEFAULT

    IDirect3DVolumeTexture9  Memory pool must be D3DPOOL_DEFAULT

    IDirect3DVertexBuffer9   Memory pool must be D3DPOOL_DEFAULT

    IDirect3DIndexBuffer9    Memory pool must be D3DPOOL_DEFAULT

   --------------------------------------------------------------------------
    Table wgl.restrictions - Restrictions on DirectX resources that can
    be registered via wglDXRegisterObjectNV
    --------------------------------------------------------------------------

    Before a GL object which is associated with a DirectX resource may
    be used, it must be locked.  The function

        BOOL wglDXLockObjectsNV(HANDLE hDevice, GLint count,
                                HANDLE *hObjects);

    attempts to lock an array of <count> interop objects.  <hObjects>
    is an array of length <count> containing the handles of the
    objects to be locked.

    A return value of TRUE indicates that all objects were
    successfully locked.  A return value of FALSE indicates an
    error. To get extended error information, call
    GetLastError. Possible errors are as follows:

      ERROR_BUSY            One or more of the objects in <hObjects>
                            was already locked.

      ERROR_INVALID_DATA    One or more of the objects in <hObjects>
                            does not belong to the interop device
                            specified by <hDevice>.

      ERROR_LOCK_FAILED     One or more of the objects in <hObjects>
                            failed to lock.

    If the function returns FALSE, none of the objects will be locked.

    Attempting to access an interop object via GL when the object is
    not locked, or attempting to access the DirectX resource through
    the DirectX API when it is locked by GL, will result in undefined
    behavior and may result in data corruption or program
    termination. Likewise, passing invalid interop device or object
    handles to this function has undefined results, including program
    termination.

    Locked objects are available for operations which read or write
    the data store, according to the access mode specified in
    wglDXRegisterObjectNV.  If a different access mode is required
    after the object has been registered, the access mode may be
    modified by calling

        BOOL wglDXObjectAccessNV(HANDLE hObject, GLenum access);

    <hObject> is an interop object handle returned by
    wglDXRegisterObjectNV and identifies the interop object for which
    the access mode should be modified.

    <access> is a new access mode with the same meaning as the
    <access> parameter of wglDXRegisterObjectNV.  The access mode may
    be modified only when an object is not locked and will affect
    subsequent lock operations.

    The return value is TRUE if the function succeeds. If an error
    occurs, the return will be FALSE. To get extended error
    information, call GetLastError. Possible errors are as follows:

      ERROR_INVALID_DATA      Invalid <access> parameter.

      ERROR_BUSY              <hObject> is currently locked for
                              GL access.

    Operations which attempt to read or write an object in a manner
    inconsistent with the specified access mode will result in
    undefined behavior and may result in data corruption or program
    termination.

    Calling wglDXObjectAccessNV with an invalid <hObject> parameter
    results in undefined behavior and may result in data corruption or
    program termination.

    In order to return control of an object to DirectX, it must be unlocked
    by calling

        BOOL wglDXUnlockObjectsNV(HANDLE hDevice, GLint count,
                                  HANDLE *hObjects);

    A return value of TRUE indicates that the objects were
    successfully unlocked and DirectX may now safely access them.  A
    return value of FALSE indicates that an error occurred. To get
    extended error information, call GetLastError. Possible errors are
    as follows:

      ERROR_NOT_LOCKED      One or more of the objects in <hObjects>
                            was not locked.

      ERROR_INVALID_DATA    One or more of the objects in <hObjects>
                            does not belong to the interop device
                            identified by <hDevice>.

      ERROR_LOCK_FAILED     One or more of the objects in <hObjects>
                            failed to unlock.

    If the function returns FALSE, none of the objects are unlocked.

    Results are undefined if any of the handles in <hObjects> are
    invalid and may result in data corruption or program termination.

    When access to a DirectX resource from GL is no longer required, the
    association between the GL object and the DirectX resource should be
    terminated by calling

        BOOL wglDXUnregisterObjectNV(HANDLE hObject);

    where <hObject> is the interop object handle returned by
    wglDXRegisterObjectNV.  Any subsequent attempt to access
    <hObject> will result in undefined behavior and may result in data
    corruption or program termination.

    A return value of TRUE indicates that the object was successfully
    unregistered and <hObject> is now invalid. A return value of FALSE
    indicates that an error occurred. To get extended error
    information, call GetLastError. Possible errors are as follows:

      ERROR_BUSY      <hObject> is currently locked for access
                      by the GL

    Results are undefined if <hObject> is invalid and may result in
    program termination or data corruption.

    When all interop operations have been completed, the connection
    between OpenGL and DirectX may be terminated by calling

        BOOL wglDXCloseDeviceNV(HANDLE hDevice);

    where <hDevice> is the interop device handle returned by
    wglDXOpenDeviceNV.  Once the device is closed, any attempt to
    access DirectX resources through associated GL handles will result
    in undefined behavior and may result in data corruption or program
    termination.

    A return value of TRUE indicates that the device was successfully
    closed and <hDevice> is now invalid. A return value of FALSE
    indicates an error. To get extended error information, call
    GetLastError. Possible errors are as follows:

        ERROR_INVALID_DATA          The Direct3D device failed to
                                    close.

    Calling this function with an invalid <hDevice> parameter results
    in undefined behavior and may result in data corruption or program
    termination.

Issues

    1) Should we support explicit usage of share handles under WDDM or
    disallow it entirely?

    RESOLUTION: We should support it. Implicit share handles are
    useful when writing code that's meant to be portable between WDDM
    and non-WDDM operating systems; explicit share handles are useful
    when writing an application that needs to share resources with
    non-GL clients.

    2) Can DirectX and OpenGL render concurrently to the same DirectX
    resource?

    RESOLUTION: Concurrent rendering by OpenGL and DirectX to the same
    resource is not supported.

    DISCUSSION: The Lock/Unlock calls serve as synchronization points
    between OpenGL and DirectX. They ensure that any rendering
    operations that affect the resource on one driver are complete
    before the other driver takes ownership of it.

    When sharing a large resource, applications can potentially desire
    concurrent access to different regions of the resource (e.g., if
    the application is drawing a user interface in DirectX with an
    OpenGL viewport occupying a region of it, where the UI and OpenGL
    regions do not overlap). In this case, more fine-grained
    synchronization could be achieved by not doing implicit
    synchronization on the driver side and providing primitives to the
    application to enable it to synchronize on it's own, for instance,
    by allowing a DirectX 9 event query to be mapped and used as a GL
    sync object. This is, however, beyond the scope of the current
    extension.

    3) If two GL contexts are sharing textures, what is the correct
    way to access a DirectX resource from both contexts?

    RESOLUTION: Sharing a DirectX resource among multiple GL contexts is
    best achieved without having shared namespaces among the GL
    contexts, by simply registering the texture on each GL context
    separately.

    If two GL contexts share namespaces, it is still necessary to lock
    the DirectX resource for each GL context that needs to access
    it. Note that only one GL context may hold the lock on the
    resource at any given time --- concurrent access from multiple GL
    contexts is not currently supported.

    4) How do driver control panel settings regarding anti-aliasing
    modes affect this extension?

    DISCUSSION: User-configurable system settings may allow users to
    force anti-aliasing for applications that do not support
    it. Usually, this causes the implementation to create multisampled
    surfaces for render targets that the application creates as
    non-multisampled.

    GL API semantics for textures can differ between multisample and
    non-multisample textures. If the application creates a DirectX
    render target that is to be bound as a GL texture, it will have no
    way to know that the surface is actually multisampled, but GL will
    require that it is bound to the TEXTURE_2D_MULTISAMPLE target
    instead of the TEXTURE_2D target. Because of this, it is
    recommended that render targets be bound to GL renderbuffers
    instead.

    Additionally, DirectX implementations are free to create render
    targets that do not match the number of samples that the app
    requested. Implementations are also free to create color and depth
    render targets with incompatible multisample modes. This can
    result in FBO completeness errors if incompatible color and depth
    render targets are bound for rendering to the same FBO. This
    problem also exists with pure DirectX applications and is not
    specific to this extension.

Sample Code

    Render to Direct3D 9 multisample color and depth buffers with
    OpenGL under WDDM:

    // create the Direct3D9Ex device:
    IDirect3D9Ex *direct3D;
    IDirect3DDevice9Ex *device;
    D3DPRESENT_PARAMETERS d3dpp;

    direct3D = Direct3DCreate9Ex(D3D_SDK_VERSION, &direct3D);

    <set appropriate device parameters in d3dpp>

    direct3D->CreateDevice(D3DADAPTER_DEFAULT,
                           D3DDEVTYPE_HAL,
                           hWnd,
                           D3DCREATE_HARDWARE_VERTEXPROCESSING |
                           D3DCREATE_PUREDEVICE |
                           D3DCREATE_MULTITHREADED,
                           &d3dpp,
                           &device);

    // create the Direct3D render targets
    IDirect3DSurface9 *dxColorBuffer;
    IDirect3DSurface9 *dxDepthBuffer;

    device->CreateRenderTarget(width, height,
                               D3DFMT_A8R8G8B8,
                               D3DMULTISAMPLE_4_SAMPLES, 0,
                               FALSE,
                               &dxColorBuffer,
                               NULL);

    device->CreateDepthStencilSurface(width, height,
                                      D3DFMT_D24S8,
                                      D3DMULTISAMPLE_4_SAMPLES, 0,
                                      FALSE,
                                      &dxDepthBuffer,
                                      NULL);

    // register the Direct3D device with GL
    HANDLE gl_handleD3D;
    gl_handleD3D = wglDXOpenDeviceNV(device);

    // register the Direct3D color and depth/stencil buffers as
    // 2D multisample textures in opengl
    GLuint gl_names[2];
    HANDLE gl_handles[2];

    glGenTextures(2, gl_names);

    gl_handles[0] = wglDXRegisterObjectNV(gl_handleD3D, dxColorBuffer,
                                          gl_names[0],
                                          GL_TEXTURE_2D_MULTISAMPLE,
                                          WGL_ACCESS_READ_WRITE_NV);

    gl_handles[1] = wglDXRegisterObjectNV(gl_handleD3D, dxDepthBuffer,
                                          gl_names[1],
                                          GL_TEXTURE_2D_MULTISAMPLE,
                                          WGL_ACCESS_READ_WRITE_NV);

    // attach the Direct3D buffers to an FBO
    glBindFramebuffer(GL_FRAMEBUFFER, fbo);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,
                           GL_TEXTURE_2D_MULTISAMPLE, gl_names[0]);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT,
                           GL_TEXTURE_2D_MULTISAMPLE, gl_names[1]);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_STENCIL_ATTACHMENT,
                           GL_TEXTURE_2D_MULTISAMPLE, gl_names[1]);

    // rendering loop
    while (!done) {
          <direct3d renders to the render targets>

          // lock the render targets for GL access
          wglDXLockObjectsNV(handleD3D, 2, gl_handles);

          <opengl renders to the render targets>

          // unlock the render targets
          wglDXUnlockObjectsNV(handleD3D, 2, gl_handles);

          <direct3d renders to the render targets and presents
           the results on the screen>
    }

Revision History

    Revision 1, 2010/11/10
     - Initial public revision