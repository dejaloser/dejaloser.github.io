---
layout: post
title: "ffmpeg를 사용하여 dxva를 사용하는 방법"
category: prog
tags: ffmpeg
keywords: ffmpeg, dxva2
---


```
FFmpeg은 코드의 DXVA 가속 디코딩을 이용하여 구현되었다. 그러나 통합 DXVA 관련된 관리 코드는 자체 플레이어에서 구현되지 않습니다.

ffmpeg구현 dxva도입 관련 코드를 디코딩

dxva2_h264.c

AVHWAccel ff_h264_dxva2_hwaccel = {  
    .name           = "h264_dxva2",  
    .type           = AVMEDIA_TYPE_VIDEO,  
    .id             = CODEC_ID_H264,  
    .pix_fmt        = PIX_FMT_DXVA2_VLD,  
    .start_frame    = start_frame,  
    .decode_slice   = decode_slice,  
    .end_frame      = end_frame,  
    .priv_data_size = sizeof(struct dxva2_picture_context),  
};

dxva2_mpeg2.c

AVHWAccel ff_mpeg2_dxva2_hwaccel = {  
    .name           = "mpeg2_dxva2",  
    .type           = AVMEDIA_TYPE_VIDEO,  
    .id             = CODEC_ID_MPEG2VIDEO,  
    .pix_fmt        = PIX_FMT_DXVA2_VLD,  
    .start_frame    = start_frame,  
    .decode_slice   = decode_slice,  
    .end_frame      = end_frame,  
    .priv_data_size = sizeof(struct dxva2_picture_context),  
};

dxva2_vc1.c

#if CONFIG_WMV3_DXVA2_HWACCEL  
AVHWAccel ff_wmv3_dxva2_hwaccel = {  
    .name           = "wmv3_dxva2",  
    .type           = AVMEDIA_TYPE_VIDEO,  
    .id             = CODEC_ID_WMV3,  
    .pix_fmt        = PIX_FMT_DXVA2_VLD,  
    .start_frame    = start_frame,  
    .decode_slice   = decode_slice,  
    .end_frame      = end_frame,  
    .priv_data_size = sizeof(struct dxva2_picture_context),  
};  
#endif

AVHWAccel ff_vc1_dxva2_hwaccel = {  
    .name           = "vc1_dxva2",  
    .type           = AVMEDIA_TYPE_VIDEO,  
    .id             = CODEC_ID_VC1,  
    .pix_fmt        = PIX_FMT_DXVA2_VLD,  
    .start_frame    = start_frame,  
    .decode_slice   = decode_slice,  
    .end_frame      = end_frame,  
    .priv_data_size = sizeof(struct dxva2_picture_context),  
};  


위의 코드를 읽고, 당신은 디코더가 외부 할당을 통해, 복제되지 dxva_context하기 위해 사용하는 것을 발견 할 것이다

struct dxva_context {  
    /** 
     * DXVA2 decoder object 
     */  
    IDirectXVideoDecoder *decoder;  
  
    /** 
     * DXVA2 configuration used to create the decoder 
     */  
    const DXVA2_ConfigPictureDecode *cfg;  
  
    /** 
     * The number of surface in the surface array 
     */  
    unsigned surface_count;  
  
    /** 
     * The array of Direct3D surfaces used to create the decoder 
     */  
    LPDIRECT3DSURFACE9 *surface;  
  
    /** 
     * A bit field configuring the workarounds needed for using the decoder 
     */  
    uint64_t workaround;  
  
    /** 
     * Private to the FFmpeg AVHWAccel implementation 
     */  
    unsigned report_id;  
};  

dxva2api.c에서, 트랜스 코딩 환경 변수 상황에 맞는 과제의 실현을 (비는 FFmpeg 소스 코드, 나에 의해 실현).

static int Setup(va_dxva2_t *va, void **hw, const AVCodecContext *avctx)  
{  
    //va_dxva2_t *va = vlc_va_dxva2_Get(external);  
    unsigned i;  
      
    if (va->width == avctx->width&& va->height == avctx->height && va->decoder)  
        goto ok;  
  
    /* */  
    DxDestroyVideoConversion(va);  
    DxDestroyVideoDecoder(va);  
  
    *hw = NULL;  
    if (avctx->width <= 0 || avctx->height <= 0)  
        return -1;  
  
    if (DxCreateVideoDecoder(va, va->codec_id, avctx))  
        return -1;  
    /* */  
    va->hw.decoder = va->decoder;  
    va->hw.cfg = &va->cfg;  
    va->hw.surface_count = va->surface_count;  
    va->hw.surface = va->hw_surface;  
    for (i = 0; i < va->surface_count; i++)  
        va->hw.surface[i] = va->surface[i].d3d;  
  
    /* */  
    DxCreateVideoConversion(va);  
  
    /* */  
ok:  
    *hw = &va->hw;  
    const d3d_format_t *output = D3dFindFormat(va->output);  
    //*chroma = output->codec;  
    return 0;  
}  

또한 환경 변수에 대해 할당 FFmpeg를 디코딩 하드웨어 솔루션 방식을 규정

if(is->iUseDxva)  
    {  
        pCodecCtx->get_buffer = DxGetFrameBuf;  
        pCodecCtx->reget_buffer = DxReGetFrameBuf;  
        pCodecCtx->release_buffer = DxReleaseFrameBuf;  
        pCodecCtx->opaque = NULL;  
        //하드웨어 솔루션의 필요성 여부
        if(pCodecCtx->codec_id == CODEC_ID_MPEG1VIDEO || pCodecCtx->codec_id == CODEC_ID_MPEG2VIDEO ||  
            //avctx->codec_id == CODEC_ID_MPEG4 ||  
            pCodecCtx->codec_id == CODEC_ID_H264 ||  
            pCodecCtx->codec_id == CODEC_ID_VC1 || pCodecCtx->codec_id == CODEC_ID_WMV3)  
        {  
            pCodecCtx->get_format = DxGetFormat;  
        }  
  
            D3DXSaveSurfaceToFile = NULL;  
        hdll = LoadLibrary(TEXT("D3DX9_42.DLL"));  
        if(hdll)  
            D3DXSaveSurfaceToFile =  (void *)GetProcAddress(hdll,TEXT("D3DXSaveSurfaceToFileA"));  
    }


FFmpeg 인터페이스 구현와 코드 dxva2 하드웨어 솔루션


/***************************************************************************** 
 * va.h: Video Acceleration API for avcodec 
 ***************************************************************************** 
 * Copyright (C) 2012 tuyuandong 
 * 
 * Authors: Yuandong Tu <tuyuandong@gmail.com> 
 * 
 * This file is part of FFmpeg.  
 *****************************************************************************/  
#include <libavutil/pixfmt.h>  
#include <libavutil/pixdesc.h>  
#include <libavcodec/avcodec.h>  
#include <va.h>  
  
enum PixelFormat DxGetFormat( AVCodecContext *avctx,  
                              const enum PixelFormat *pi_fmt )  
{  
    unsigned int i;  
    dxva_t *p_va = (dxva_t*)avctx->opaque;  
  
    if( p_va != NULL )  
        dxva_Delete( p_va );  
  
    p_va = dxva_New(avctx->codec_id);  
    if(p_va != NULL )  
    {  
        /* Try too look for a supported hw acceleration */  
        for(i = 0; pi_fmt[i] != PIX_FMT_NONE; i++ )  
        {  
            const char *name = av_get_pix_fmt_name(pi_fmt[i]);  
            av_log(NULL,AV_LOG_DEBUG, "Available decoder output format %d (%s)",  
                     pi_fmt[i], name ? name : "unknown" );  
            if( p_va->pix_fmt != pi_fmt[i] )  
                continue;  
  
            /* We try to call dxva_Setup when possible to detect errors when 
             * possible (later is too late) */  
            if( avctx->width > 0 && avctx->height > 0  
             && dxva_Setup(p_va, &avctx->hwaccel_context,avctx) )  
            {  
                av_log(NULL,AV_LOG_ERROR, "acceleration setup failure" );  
                break;  
            }  
  
            //if( p_va->description )  
            //    av_log(NULL,AV_LOG_INFO, "Using %s for hardware decoding.",  
            //              p_va->description );  
  
            /* FIXME this will disable direct rendering 
             * even if a new pixel format is renegotiated 
             */  
            //p_sys->b_direct_rendering = false;  
            //p_sys->p_va = p_va;  
            avctx->opaque = p_va;  
            avctx->draw_horiz_band = NULL;             
            return pi_fmt[i];  
        }  
  
        av_log(NULL,AV_LOG_ERROR, "acceleration not available" );  
        dxva_Delete( p_va );  
    }  
    avctx->opaque = NULL;  
    /* Fallback to default behaviour */  
    return avcodec_default_get_format(avctx, pi_fmt );  
}  

/***************************************************************************** 
 * DxGetFrameBuf: callback used by ffmpeg to get a frame buffer. 
 ***************************************************************************** 
 * It is used for direct rendering as well as to get the right PTS for each 
 * decoded picture (even in indirect rendering mode). 
 *****************************************************************************/  

int DxGetFrameBuf( struct AVCodecContext *avctx,  
                               AVFrame *pic )  
{  
    dxva_t *p_va = (dxva_t *)avctx->opaque;  
    //picture_t *p_pic;  
  
    /* */  
    pic->reordered_opaque = avctx->reordered_opaque;  
    pic->opaque = NULL;  
  
    if(p_va)  
    {  
        /* hwaccel_context is not present in old ffmpeg version */  
        if( dxva_Setup(p_va,&avctx->hwaccel_context, avctx) )  
        {  
            av_log(NULL,AV_LOG_ERROR, "vlc_va_Setup failed" );  
            return -1;  
        }  
  
        /* */  
        pic->type = FF_BUFFER_TYPE_USER;  
  
#if LIBAVCODEC_VERSION_MAJOR < 54  
        pic->age = 256*256*256*64;  
#endif  
  
        if(dxva_Get(p_va,pic ) )  
        {  
            av_log(NULL,AV_LOG_ERROR, "VaGrabSurface failed" );  
            return -1;  
        }  
        return 0;  
    }  
  
    return avcodec_default_get_buffer(avctx,pic );  
}  
int  DxReGetFrameBuf( struct AVCodecContext *avctx, AVFrame *pic )  
{  
    pic->reordered_opaque = avctx->reordered_opaque;  
  
    /* We always use default reget function, it works perfectly fine */  
    return avcodec_default_reget_buffer(avctx, pic );  
}  
  
void DxReleaseFrameBuf( struct AVCodecContext *avctx,  
                                    AVFrame *pic )  
{  
    dxva_t *p_va = (dxva_t *)avctx->opaque;  
    int i;  
      
    if(p_va )  
    {  
        dxva_Release(p_va, pic );  
    }  
    else if( !pic->opaque )  
    {  
        /* We can end up here without the AVFrame being allocated by 
         * avcodec_default_get_buffer() if VA is used and the frame is 
         * released when the decoder is closed 
         */  
        if( pic->type == FF_BUFFER_TYPE_INTERNAL )  
            avcodec_default_release_buffer( avctx,pic );  
    }  
    else  
    {  
        //picture_t *p_pic = (picture_t*)p_ff_pic->opaque;  
        //decoder_UnlinkPicture( p_dec, p_pic );  
        av_log(NULL,AV_LOG_ERROR,"%d %s a error is rasied\e\n");  
    }  
    for(i = 0; i < 4; i++ )  
        pic->data[i] = NULL;  
}  
  
int DxPictureCopy(struct AVCodecContext *avctx,AVFrame *src, AVFrame* dst)  
{  
    dxva_t *p_va = (dxva_t *)avctx->opaque;  
    return dxva_Extract(p_va,src,dst);  
}  


/***************************************************************************** 
 * OnlyVideo.c: Video Decode Test 
 ***************************************************************************** 
 * Copyright (C) 2012 tuyuandong 
 * 
 * Authors: Yuandong Tu <tuyuandong@gmail.com> 
 * 
 * Data 2012-11-28 
 * 
 * This file is part of FFmpeg.  
 *****************************************************************************/  
#include <windows.h>  
#include <d3d9.h>  
#include <stdio.h>  
#include "libavcodec/avcodec.h"  
#include "libavformat/avformat.h"  
#include "va.h"  
  
typedef struct VideoState {  
    int     abort_request;  
    char        filename[1024];  
    int     srcWidth,srcHeight;
    int     iUseDxva;  
}VideoState;  
char        out_path[128];  
  
void help();  
DWORD VideoDecodec(LPVOID lpParameter);  
  
int main(int argc, char *argv[])  
{  
    VideoState is;  
    int i;  
    if(argc <2 || (argc == 2 && !strcmp(argv[1],"--help")))  
    {  
        help();  
        return 0;  
    }  
  
    memset(&is,0,sizeof(is));  
    memset(out_path,0,sizeof(out_path));  
    sprintf(is.filename,"%s",argv[1]);  
    if(argc > 2 && !strcmp(argv[2],"-dxva"))  
        is.iUseDxva = 1;      
    if(argc >4  && !strcmp(argv[3],"-o"))  
        strcpy(out_path,argv[4]);  
      
    if(out_path[0])  
        CreateDirectory(out_path,NULL);  
      
    VideoDecodec(&is);  
    return 0;  
}  
  
void help()  
{  
    printf("**********************************************\n");  
    printf("Usage:\n");  
    printf("    OnlyVideo file [options]\n");  
    printf("\n");  
    printf("Options: \n");  
    printf("    -dxva       \n");  
    printf("    -o out_path\n");  
    printf("\n");  
    printf("Examples: \n");  
    printf("    OnlyVideo a.mpg -dxva\n");  
    printf("**********************************************\n");    
}  
  
void console_log(const char *fmt, ...)  
{  
    HANDLE h;  
    va_list vl;  
    va_start(vl, fmt);  
  
    char buf[1024];  
    memset(buf,0,1024);  
    vsprintf(buf,fmt,vl);  
  
    h = GetStdHandle(STD_OUTPUT_HANDLE);  
    COORD pos;  
    ULONG unuse;  
    pos.X= 0;  
    CONSOLE_SCREEN_BUFFER_INFO bInfo;
    GetConsoleScreenBufferInfo(h, &bInfo );  
    pos.Y= bInfo.dwCursorPosition.Y;  
    WriteConsoleOutputCharacter(h,buf,strlen(buf),pos,&unuse);  
      
    va_end(vl);   
}  
  
  
typedef enum _D3DXIMAGE_FILEFORMAT  
{  
    D3DXIFF_BMP         = 0,  
    D3DXIFF_JPG         = 1,  
    D3DXIFF_TGA         = 2,  
    D3DXIFF_PNG         = 3,  
    D3DXIFF_DDS         = 4,  
    D3DXIFF_PPM         = 5,  
    D3DXIFF_DIB         = 6,  
    D3DXIFF_HDR         = 7,       //high dynamic range formats  
    D3DXIFF_PFM         = 8,       //  
    D3DXIFF_FORCE_DWORD = 0x7fffffff  
  
} D3DXIMAGE_FILEFORMAT;  
  
void SaveFrame(AVFrame *pFrame, int width, int height, int iFrame) {  
    FILE *pFile;  
    char szFilename[32];  
    int  y;  
  
    // Open file  
    if(out_path[0])  
        sprintf(szFilename, ".\\%s\\frame%d.ppm",out_path, iFrame);  
    else  
        sprintf(szFilename, ".\\frame%d.ppm", iFrame);  
      
    pFile=fopen(szFilename, "wb");  
    if(pFile==NULL)  
        return;  
  
    // Write header  
    fprintf(pFile, "P6\n%d %d\n255\n", width, height);  
  
    // Write pixel data  
    for(y=0; y<height; y++)  
        fwrite(pFrame->data[0]+y*pFrame->linesize[0], 1, width*3, pFile);  
  
    // Close file  
    fclose(pFile);  
}  
  
DWORD VideoDecodec(LPVOID lpParameter)  
{  
DWORD  (*D3DXSaveSurfaceToFile)(  
        char*                    pDestFile,  
        D3DXIMAGE_FILEFORMAT      DestFormat,  
        LPDIRECT3DSURFACE9        pSrcSurface,  
        CONST PALETTEENTRY*       pSrcPalette,  
        CONST RECT*               pSrcRect);  
  
    VideoState*     is = lpParameter;  
    AVFormatContext *pFormatCtx;  
    int             i, videoStream;  
    AVCodecContext  *pCodecCtx;  
    AVCodec         *pCodec;  
    AVFrame         *pFrame;   
    AVFrame        *pFrameYUV = NULL;   
    char*       pYUVBuf =NULL;  
    AVPacket        pkt;  
    int             frames;  
    DWORD           dwTicks,cost;  
    DWORD           sec;  
    HINSTANCE   hdll = NULL;  
  
      
    // Register all formats and codecs  
    av_register_all();  
  
    // Open video file  
    if(av_open_input_file(&pFormatCtx, is->filename, NULL, 0, NULL)!=0)  
        return -1; // Couldn't open file  
  
      
    // Retrieve stream information  
    //if(av_find_stream_info(pFormatCtx)<0)  
    //  return -1; // Couldn't find stream information  
      
    // Dump information about file onto standard error  
    dump_format(pFormatCtx, 0, is->filename, 0);  
  
      
    // Find the first video stream  
    videoStream=-1;  
    for(i=0; i<pFormatCtx->nb_streams; i++)  
        if(pFormatCtx->streams[i]->codec->codec_type==AVMEDIA_TYPE_VIDEO)  
    {  
        videoStream=i;  
        break;  
    }  
      
    if(videoStream==-1)  
        return -1; // Didn't find a video stream  
  
    // Get a pointer to the codec context for the video stream  
    pCodecCtx=pFormatCtx->streams[videoStream]->codec;  
  
    pCodec=avcodec_find_encoder(CODEC_ID_H264);  
  
    // Find the decoder for the video stream  
    pCodec=avcodec_find_decoder(pCodecCtx->codec_id);  
    if(pCodec==NULL) {  
        fprintf(stderr, "Unsupported codec!\n");  
        return -1; // Codec not found  
    }  
  
    if(is->iUseDxva)  
    {  
        pCodecCtx->get_buffer = DxGetFrameBuf;  
        pCodecCtx->reget_buffer = DxReGetFrameBuf;  
        pCodecCtx->release_buffer = DxReleaseFrameBuf;  
        pCodecCtx->opaque = NULL;  
        //是否为需要硬解  
        if(pCodecCtx->codec_id == CODEC_ID_MPEG1VIDEO || pCodecCtx->codec_id == CODEC_ID_MPEG2VIDEO ||  
            //avctx->codec_id == CODEC_ID_MPEG4 ||  
            pCodecCtx->codec_id == CODEC_ID_H264 ||  
            pCodecCtx->codec_id == CODEC_ID_VC1 || pCodecCtx->codec_id == CODEC_ID_WMV3)  
        {  
            pCodecCtx->get_format = DxGetFormat;  
        }  
  
            D3DXSaveSurfaceToFile = NULL;  
        hdll = LoadLibrary(TEXT("D3DX9_42.DLL"));  
        if(hdll)  
            D3DXSaveSurfaceToFile =  (void *)GetProcAddress(hdll,TEXT("D3DXSaveSurfaceToFileA"));  
    }  
          
    // Open codec  
    if(avcodec_open(pCodecCtx, pCodec)<0)  
        return -1; // Could not open codec  
  
  
    // Allocate video frame  
    pFrame=avcodec_alloc_frame();  
    pFrameYUV = avcodec_alloc_frame();  
    if(pFrameYUV && is->iUseDxva)  
    {  
        int numBytes=avpicture_get_size(PIX_FMT_YUV420P,pCodecCtx->width,pCodecCtx->height);  
        pYUVBuf=(uint8_t *)av_malloc(numBytes*sizeof(uint8_t));  
        pFrameYUV->width = pCodecCtx->width;  
        pFrameYUV->height =pCodecCtx->height;  
        avpicture_fill((AVPicture *)pFrameYUV, pYUVBuf, PIX_FMT_YUV420P,pCodecCtx->width,pCodecCtx->height);  
    }  
      
  
    frames=0;  
    cost = 0;  
    sec = 0;  
    while(!is->abort_request && av_read_frame(pFormatCtx, &pkt)>=0) {  
        // Is this a packet from the video stream?  
        if(pkt.stream_index==videoStream) {  
            int got_picture;  
            // Decode video frame  
            dwTicks = GetTickCount();  
            avcodec_decode_video2(pCodecCtx, pFrame, &got_picture,&pkt);  
            cost += (GetTickCount() - dwTicks);  
              
            // Did we get a video frame?  
            if(got_picture) {     
                frames++;  
                if(cost/1000 > sec)  
                {  
                    float speeds= (frames*1000.0)/cost;  
                    console_log("pic_type[%d] frames[%d] times[%d] speeds[%f fps]\n",  
                        pFrame->format,frames,cost,speeds);  
                    sec ++;  
  
                    if(pFrame->format == PIX_FMT_DXVA2_VLD )  
                    {  
                        //char filename[256] = {0};  
                        //DWORD ret = 0;  
                        //LPDIRECT3DSURFACE9 d3d = pFrame->data[3];  
                        //sprintf(filename,"e://%d.jpg",frames);  
                        //if(D3DXSaveSurfaceToFile)  
                        //  ret = D3DXSaveSurfaceToFile(filename,D3DXIFF_JPG/*jpeg*/,d3d,NULL,NULL);  
                        //filename[128] =  0;  
                        DxPictureCopy(pCodecCtx,pFrame,pFrameYUV);  
                        SaveFrame(pFrameYUV,pFrameYUV->width,pFrameYUV->height,frames);  
                    }  
                }  
            }  
        }  
        // Free the packet that was allocated by av_read_frame  
        av_free_packet(&pkt);  
    }  
  
    avcodec_close(pCodecCtx);  
    //av_close_input_file(pFormatCtx);  
    if(pYUVBuf)  
        av_free(pYUVBuf);  
    if(pFrame)  
        av_free(pFrame);  
    if(pFrameYUV)  
        av_free(pFrameYUV);  
    if(hdll)  
        FreeLibrary(hdll);  
}
```

http://download.csdn.net/detail/tttyd/5166123
http://pan.baidu.com/share/link?shareid=703510495&uk=3240542740