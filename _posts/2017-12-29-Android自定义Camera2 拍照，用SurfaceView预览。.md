---
layout: post
title:  "Android�Զ���Camera2 ���գ���SurfaceViewԤ����"
subtitle:  "�Զ���Camera2"
date:  2017-12-29 
author:  "Mtj"
 tags:
     Android
     SurfaceView
     �Զ���camera
2     
---

* �����ļ�����˵�ˣ�������SurfaceView������������ڵĲ��֣��Լ����żӰɣ�

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
* surfaceView��findViewById()���¾Ͳ�˵�ˣ�ֱ�ӿ�ʼԤ���Ĵ���

```
    //6.0��ʼ
    private CameraManager manager;
    private Handler childHandler, mainHandler;
    private CameraDevice mCamera;
    private CaptureRequest.Builder mPreviewBuilder;
    private CameraCaptureSession mSession;
    private ImageReader mImageReader;
    // ����������Ҫ��CaptureRequest.Builder
    private CaptureRequest.Builder captureRequestBuilder;
    //6.0����
    
    //�ܶ���̶�������첽���ˣ�����������Ҫһ�����̵߳�looper
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
                //��Ҫ���Ȩ��
                    if (ActivityCompat.checkSelfPermission(getPageContext(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                        return;
                    }
                    //��ȡ��������豸�б�
                    String[] CameraIdList = manager.getCameraIdList();
                    //�����
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
                //�ͷ���Դ
                if (mCamera != null) {
                    mCamera.close();
                    mCamera = null;
                }
            }
        });
        //������Ƭ�Ĵ�С
        mImageReader = ImageReader.newInstance(3264, 1840, ImageFormat.JPEG, 2);
        mImageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
            @Override
            public void onImageAvailable(ImageReader imageReader) {
                // �õ�������Ƭ����
                Image image = imageReader.acquireNextImage();
                ByteBuffer buffer = image.getPlanes()[0].getBuffer();
                byte[] bytes = new byte[buffer.remaining()];
                buffer.get(bytes);//�ɻ����������ֽ�����
                image.close();
                //saveBitmap(bytes);//������Ƭ�Ĵ���
            }
        }, mainHandler);


/**
     * ����ͷ��������
     */
    private CameraDevice.StateCallback mCameraDeviceStateCallback = new CameraDevice.StateCallback() {
        @Override
        public void onOpened(CameraDevice camera) {//������ͷ
            try {
                //����Ԥ��
                mCamera = camera;
                startPreview(camera);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onDisconnected(CameraDevice camera) {
            //�ر�����ͷ
            if (mCamera != null) {
                mCamera.close();
                mCamera = null;
            }
        }

        @Override
        public void onError(CameraDevice camera, int error) {
            //��������
        }
    };

    //��ʼԤ������Ҫ��camera.createCaptureSession��δ������Ҫ�������Ự
    private void startPreview(final CameraDevice camera) throws CameraAccessException {
        try {
            // ����Ԥ����Ҫ��CaptureRequest.Builder
            mPreviewBuilder = camera.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            // ��SurfaceView��surface��ΪCaptureRequest.Builder��Ŀ��
            mPreviewBuilder.addTarget(holder.getSurface());
            mPreviewBuilder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_OFF);
            mPreviewBuilder.set(CaptureRequest.CONTROL_AE_MODE, CaptureRequest.CONTROL_AE_MODE_OFF);
            //��������ͼ��ʱ����豸�Ƿ�ʹ�ù�ѧ������OIS����
            mPreviewBuilder.set(CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE, CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE_ON);
            //�й�������
            mPreviewBuilder.set(CaptureRequest.SENSOR_SENSITIVITY, 1600);
            //�عⲹ��//
            mPreviewBuilder.set(CaptureRequest.CONTROL_AE_EXPOSURE_COMPENSATION, 0);
            // ����CameraCaptureSession���ö����������Ԥ���������������
            camera.createCaptureSession(Arrays.asList(holder.getSurface(), mImageReader.getSurface()), mSessionStateCallback, childHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    /**
     * �Ự״̬�ص�
     */
    private CameraCaptureSession.StateCallback mSessionStateCallback = new CameraCaptureSession.StateCallback() {
        @Override
        public void onConfigured(CameraCaptureSession session) {
            mSession = session;
            if (mCamera != null && captureRequestBuilder == null) {
                try {
                    captureRequestBuilder = mCamera.createCaptureRequest(CameraDevice.TEMPLATE_STILL_CAPTURE);
                    // ��imageReader��surface��ΪCaptureRequest.Builder��Ŀ��
                    captureRequestBuilder.addTarget(mImageReader.getSurface());
                    //�ر��Զ��Խ�
                    captureRequestBuilder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_OFF);
                    captureRequestBuilder.set(CaptureRequest.CONTROL_AE_MODE, CaptureRequest.CONTROL_AE_MODE_OFF);
                    //��������ͼ��ʱ����豸�Ƿ�ʹ�ù�ѧ������OIS����
                  captureRequestBuilder.set(CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE, CaptureRequest.LENS_OPTICAL_STABILIZATION_MODE_ON);
                    captureRequestBuilder.set(CaptureRequest.SENSOR_SENSITIVITY, valueISO);
                    //�عⲹ��//
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
     * ���»Ự������Ԥ��
     *
     * @param session
     * @throws CameraAccessException
     */
    private void updatePreview(CameraCaptureSession session) throws CameraAccessException {
        session.setRepeatingRequest(mPreviewBuilder.build(), mCaptureCallback, childHandler);
    }
```
*  Ԥ���Ѿ�OK�ˡ��������ɡ�
* ������һ��camera���յķ����� 

```
    /**
     * ������Ƭ
     */
    private void takePicture() {
        if (mCamera == null) {
            return;
        }
        if (mSession != null && captureRequestBuilder != null) {
            //����
            try {
                CaptureRequest cr = captureRequestBuilder.build();
                mSession.capture(cr, null, null);//����API��Ҳ���Ե����ĵ�Ŷ
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }
    }
```

* OK����Ҳ�����ˣ��������ǵ�����������ƵĿ����͹ر���
```
/**
     * 6.0��
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
     * 6.0��
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
* OK���Զ���camera2 ����Ԥ��������ˡ�

