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
                if (mCamera != null) {
                    try {
                        mCamera.setPreviewDisplay(holder);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }

            @Override
            public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {
                if (mCamera != null) {
                    Camera.Parameters parameters = mCamera.getParameters();
                    //设置测光区域
                    if (parameters.getMaxNumMeteringAreas() > 0) {
                        Rect meteringRect = new Rect(-1000, -500, 1000, 100);
                        List<Camera.Area> mMeteringList = new ArrayList<>();
                        mMeteringList.add(new Camera.Area(meteringRect, 1000));
                        parameters.setMeteringAreas(mMeteringList);
                    }
                    //设置感光度ISO
                    IsoBean b = getIso(parameters);
                    parameters.set(b.getKey(), b.getIso());
                    //获取当前模式支持的感光度
                    // 选择合适的图片尺寸，必须是手机支持的尺寸
                    List<Camera.Size> sizeList = parameters.getSupportedPictureSizes();
                    // 如果sizeList只有一个我们也没有必要做什么了，因为就他一个别无选择
                    if (sizeList.size() > 1) {
                        for (int j = 0; j < sizeList.size(); j++) {
                            Camera.Size size = sizeList.get(j);//ConstantParam.picture_size
                            if (size.height >= 720 && size.height <= ConstantParam.picture_size && size.height >= PreviewHeight) {
                                PreviewWidth = size.width;
                                PreviewHeight = size.height;
                            }
                        }
                    }
                    //设置曝光补偿
                    parameters.setExposureCompensation(-3);
                    //设置照片的大小
                    parameters.setPictureSize(PreviewWidth, PreviewHeight);
                    try {
                        //设置预览的大小，必须是手机支持的大小。这里为了方便直接try,catch去设置了。别说懒...
                        parameters.setPreviewSize(1280, 720);
                    } catch (Exception e) {
                    }
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

    //这个方法是目前针对camera设置ISO比较好的方法。我也是拷贝来滴。就是这么懒...
    public IsoBean getIso(Camera.Parameters params) {
        String flat = params.flatten();
        //Log.i("flat", flat);
        String[] isoValues = null;
        String values_keyword = null;
        String iso_keyword = null;
        if (flat.contains("iso-mode-values")) {
            // google galaxy nexus keywords
            values_keyword = "iso-mode-values";
            iso_keyword = "iso";
        } else if (flat.contains("iso-speed-values")) {
            // micromax a101 keywords
            values_keyword = "iso-speed-values";
            iso_keyword = "iso-speed";
        } else if (flat.contains("nv-picture-iso-values")) {
            // LG dual p990 keywords
            values_keyword = "nv-picture-iso-values";
            iso_keyword = "nv-picture-iso";
        } else if (flat.contains("iso-values")) {
            // most used keywords
            values_keyword = "iso-values";
            iso_keyword = "iso";
        }
        // add other eventual keywords here...
        if (iso_keyword != null) {
            // flatten contains the iso key!!
            String iso = flat.substring(flat.indexOf(values_keyword));
            iso = iso.substring(iso.indexOf("=") + 1);
            if (iso.contains(";")) {
                iso = iso.substring(0, iso.indexOf(";"));
            }
            isoValues = iso.split(",");
            IsoBean bean = new IsoBean();
            bean.setKey(iso_keyword);
            bean.setValues(isoValues);
            return bean;
        }
        return null;
    }

/**
     * 获取打开相机
     *
     * @return
     */
    private Camera getCustomCamera() {
        if (null == mCamera) {
            //使用Camera的Open函数开机摄像头硬件
            //Camera.open()方法说明：2.3以后支持多摄像头，所以开启前可以通过getNumberOfCameras先获取摄像头数目，
            // 再通过 getCameraInfo得到需要开启的摄像头id，然后传入Open函数开启摄像头，
            // 假如摄像头开启成功则返回一个Camera对象
            try {
                mCamera = Camera.open();
                //预览画面旋转90度
                mCamera.setDisplayOrientation(90);
            } catch (Exception e) {
            }
        }
        return mCamera;
    }
```
* 其中设置感光度ISO的实体类如下： 

```
public class IsoBean implements Parcelable {
    private String key;
    private String[] values;
    private String iso = "auto";

    public String getIso() {
        if (values != null && values.length > 0) {
        //获取最大的感光度
            iso = values[values.length - 1];
        }
        return iso;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public String[] getValues() {
        return values;
    }

    public void setValues(String[] values) {
        this.values = values;
    }


    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.key);
        dest.writeStringArray(this.values);
    }

    public IsoBean() {
    }

    protected IsoBean(Parcel in) {
        this.key = in.readString();
        this.values = in.createStringArray();
    }

    public static final Creator<IsoBean> CREATOR = new Creator<IsoBean>() {
        @Override
        public IsoBean createFromParcel(Parcel source) {
            return new IsoBean(source);
        }

        @Override
        public IsoBean[] newArray(int size) {
            return new IsoBean[size];
        }
    };
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
