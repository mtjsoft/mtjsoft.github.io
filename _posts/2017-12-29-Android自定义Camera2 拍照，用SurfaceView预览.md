---
layout: post
title:  "Android自定义Camera2拍照，用SurfaceView预览"
subtitle:  "自定义Camera2"
date:  2017-12-29 
author:  "Mtj"
 tags:
     Android
     SurfaceView
     自定义camera2    
     
---

* 布局文件不用说了，就它了SurfaceView。其他花里古哨的布局，自己想着加吧！

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <SurfaceView
        android:id="@+id/surfaceview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```
* surfaceView的findViewById()的事就不说了，直接开始预览的代码

```
    //6.0开始
    private CameraManager manager;
    private Handler childHandler, mainHandler;
    private CameraDevice mCamera;
    private CaptureRequest.Builder mPreviewBuilder;
    private CameraCaptureSession mSession;
    private ImageReader mImageReader;
    // 创建拍照需要的CaptureRequest.Builder
    private CaptureRequest.Builder captureRequestBuilder;
    //6.0结束
    
    //很多过程都变成了异步的了，所以这里需要一个子线程的looper
    HandlerThread handlerThread = new HandlerThread("Camera2");
    handlerThread.start();
    childHandler = new Handler(handlerThread.getLooper());
    mainHandler = new Handler(getMainLooper());
    manager = (CameraManager) getSystemService(Context.CAMERA_SERVICE);
        holder = surfaceView.getHolder();
        holder.setKeepScreenOn(true);
        holder.addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder surfaceHolder) {
                try {
                //需要相机权限
                    if (ActivityCompat.checkSelfPermission(getPageContext(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                        return;
                    }
                    //获取可用相机设备列表
                    String[] CameraIdList = manager.getCameraIdList();
                    //打开相机
                    manager.openCamera(CameraIdList[0], mCameraDeviceStateCallback, mainHandler);
                } catch (CameraAccessException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {

            }

            @Override
            public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
                //释放资源
                if (mCamera != null) {
                    mCamera.close();
                    mCamera = null;
                }
            }
        });
        //设置照片的大小
        mImageReader = ImageReader.newInstance(3264, 1840, ImageFormat.JPEG, 2);
        mImageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
            @Override
            public void onImageAvailable(ImageReader imageReader) {
                // 拿到拍照照片数据
                Image image = imageReader.acquireNextImage();
                ByteBuffer buffer = image.getPlanes()[0].getBuffer();
                byte[] bytes = new byte[buffer.remaining()];
                buffer.get(bytes);//由缓冲区存入字节数组
                image.close();
                //saveBitmap(bytes);//保存照片的处理
            }
        }, mainHandler);


/**
     * 摄像头创建监听
     */
    private CameraDevice.StateCallback mCameraDeviceStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(CameraDevice camera) {//打开摄像头
            try {
                //开启预览
                mCamera = camera;
                startPreview(camera);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onDisconnected(CameraDevice camera) {
            //关闭摄像头
            if (mCamera != null) {
                mCamera.close();
                mCamera = null;
            }
        }

        @Override
        public void onError(CameraDevice camera, int error) {
            //发生错误
        }
    };

    //开始预览，主要是camera.createCaptureSession这段代码很重要，创建会话
    private void startPreview(final CameraDevice camera) throws CameraAccessException {
        try {
            // 创建预览需要的CaptureRequest.Builder
            mPreviewBuilder = camera.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            // 将SurfaceView的surface作为CaptureRequest.Builder的目标
            mPreviewBuilder.addTarget(holder.getSurface());
            mPreviewBuilder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_OFF);
            mPreviewBuilder.set(CaptureRequest.CONTROL_AE_MODE, CaptureRequest.CONTROL_AE_MODE_OFF);
            //设置拍摄图像时相机设备是否使用光学防抖（OIS）。
            mPreviewBuilder.set(CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE, CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE_ON);
            //感光灵敏度
            mPreviewBuilder.set(CaptureRequest.SENSOR_SENSITIVITY, 1600);
            //曝光补偿//
            mPreviewBuilder.set(CaptureRequest.CONTROL_AE_EXPOSURE_COMPENSATION, 0);
            // 创建CameraCaptureSession，该对象负责管理处理预览请求和拍照请求
            camera.createCaptureSession(Arrays.asList(holder.getSurface(), mImageReader.getSurface()), mSessionStateCallback, childHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    /**
     * 会话状态回调
     */
    private CameraCaptureSession.StateCallback mSessionStateCallback = new CameraCaptureSession.StateCallback() {
        @Override
        public void onConfigured(CameraCaptureSession session) {
            mSession = session;
            if (mCamera != null && captureRequestBuilder == null) {
                try {
                    captureRequestBuilder = mCamera.createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
                    // 将imageReader的surface作为CaptureRequest.Builder的目标
                    captureRequestBuilder.addTarget(mImageReader.getSurface());
                    //关闭自动对焦
                    captureRequestBuilder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_OFF);
                    captureRequestBuilder.set(CaptureRequest.CONTROL_AE_MODE, CaptureRequest.CONTROL_AE_MODE_OFF);
                    //设置拍摄图像时相机设备是否使用光学防抖（OIS）。
                  captureRequestBuilder.set(CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE, CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE_ON);
                    captureRequestBuilder.set(CaptureRequest.SENSOR_SENSITIVITY, valueISO);
                    //曝光补偿//
                 captureRequestBuilder.set(CaptureRequest.CONTROL_AE_EXPOSURE_COMPENSATION, 0);
                } catch (CameraAccessException e) {
                    e.printStackTrace();
                }
            }
            try {
                updatePreview(session);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onConfigureFailed(CameraCaptureSession session) {

        }
    };

/**
     * 更新会话，开启预览
     *
     * @param session
     * @throws CameraAccessException
     */
    private void updatePreview(CameraCaptureSession session) throws CameraAccessException {
        session.setRepeatingRequest(mPreviewBuilder.build(), mCaptureCallback, childHandler);
    }
```
*  预览已经OK了。简单至极吧。
* 下面是一个camera拍照的方法： 

```
    /**
     * 单拍照片
     */
    private void takePicture() {
        if (mCamera == null) {
            return;
        }
        if (mSession != null && captureRequestBuilder != null) {
            //拍照
            try {
                CaptureRequest cr = captureRequestBuilder.build();
                mSession.capture(cr, null, null);//单拍API，也可以调连拍的哦
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }
    }
```

* OK拍照也可以了，再来就是单独控制闪光灯的开启和关闭了

```
    /**
     * 6.0开
     */
    public void openLight6() {
        try {
            mPreviewBuilder.set(CaptureRequest.FLASH_MODE,
                    CaptureRequest.FLASH_MODE_TORCH);
            updatePreview(mSession);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    /**
     * 6.0关
     */
    public void closeLight6() {
        try {
            mPreviewBuilder.set(CaptureRequest.FLASH_MODE,
                    CaptureRequest.FLASH_MODE_OFF);
            updatePreview(mSession);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }
```

* OK，自定义camera2 拍照预览就完成了。

