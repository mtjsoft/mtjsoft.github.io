---
layout: post
title:  "Android自定义Camera，用SurfaceView预览"
subtitle:  "自定义Camera"
date:  2017-11-04
author:  "Mtj"
tags:
     Android
     SurfaceView
     自定义camera
     
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
/**
     * 开始预览
     */
    private void startPreview() {
        holder = surfaceView.getHolder();
        holder.addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder surfaceHolder) {
                mCamera = getCustomCamera();
            }

            @Override
            public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {
                if (mCamera != null) {
                    Camera.Parameters parameters = mCamera.getParameters();
                    // 选择合适的图片尺寸，必须是手机支持的尺寸
                    List<Camera.Size> sizeList = parameters.getSupportedPictureSizes();
                    // 如果sizeList只有一个我们也没有必要做什么了，因为就他一个别无选择
                    if (sizeList.size() > 1) {
                        for (int j = 0; j < sizeList.size(); j++) {
                            Camera.Size size = sizeList.get(j);
                                previewWidth = size.width;
                                previewHeight = size.height;
                        }
                    }
                    //设置照片的大小
                    parameters.setPictureSize(previewWidth , previewHeight );
                    mCamera.setParameters(parameters);
                    try {
                        mCamera.setPreviewDisplay(holder);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    //调用相机预览功能
                    mCamera.startPreview();
                }
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
                if (null != mCamera) {
                    //停止预览
                    mCamera.stopPreview();
                    //释放相机资源
                    mCamera.release();
                    mCamera = null;
                }
            }
        });
    }

    /**
     * 获取打开相机
     *
     * @return
     */
    private Camera getCustomCamera() {
        if (null == mCamera) {
            //Camera.open()方法说明：2.3以后支持多摄像头，所以开启前可以通过getNumberOfCameras先获取摄像头数目，
            // 再通过 getCameraInfo得到需要开启的摄像头id，然后传入Open函数开启摄像头，
            // 假如摄像头开启成功则返回一个Camera对象
            try {
                mCamera = Camera.open();
                //预览画面默认是横屏的，需要旋转90度
                mCamera.setDisplayOrientation(90);
            } catch (Exception e) {
            }
        }
        return mCamera;
    }
```

* 预览已经OK了。简单至极吧。
*  下面是一个camera拍照的方法：

```
mCamera.takePicture(null, null, new Camera.PictureCallback() {
                @Override
                public void onPictureTaken(byte[] data, Camera camera) {
                    if (mCamera != null) {
                    //在拍照完成时，重新开始预览
                        mCamera.startPreview();
                        //转换成bitmap 
                        Bitmap bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);
                    }
                }
            });
```
* OK拍照也可以了，再来就是单独控制闪光灯的开启和关闭了

```
if (mCamera != null) {
            try {
                //打开闪光灯
                Camera.Parameters parameter = mCamera.getParameters();
                parameter.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);
                mCamera.setParameters(parameter);
            } catch (Exception e) {
            }
        }



if (mCamera != null) {
            try {
                //关闭闪光灯
                Camera.Parameters parameter = mCamera.getParameters();
                parameter.setFlashMode(Camera.Parameters.FLASH_MODE_OFF);
                mCamera.setParameters(parameter);
            } catch (Exception e) {
            }
        }
```
**最后在Activity销毁时，记得要释放相机资源**

```
@Override
    public void onStop() {
        super.onStop();
        if (null != mCamera) {
            //停止预览
            mCamera.stopPreview();
            //释放相机资源
            mCamera.release();
            mCamera = null;
        }
    }
```
**OK，自定义camera就完成了。后面会出一篇camera2的使用方法。相比camera，camera2还是比较好使的。**
