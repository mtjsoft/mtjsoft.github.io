---
layout: post
title:  "Recyclerview֧������ˢ�¡��������أ��������Բ��֡����񲼾ֺ� ��ʽ����"
subtitle:  "Recyclerview֧������ˢ�¡��������أ��������Բ��֡����񲼾ֺ� ��ʽ����"
date:  2017-06-26 15:00
author:  "Mtj"
tags:
     Android
     Recyclerview
     �ٲ���
---

# SwiperefreshRecyclerview
## ֧������ˢ�¡��������ص� Recyclerview���������Բ��֡����񲼾ֺ� �����֡�
##ֻ��Ҫ��adapter�а����ݣ������Ľ����ң�������ô6��

### To get a Git project into your build:
### Step 1. Add the JitPack repository to your build file 
### Add it in your root build.gradle at the end of repositories:
```
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}      
```
## Step 2. Add the dependency
```
dependencies {     
  compile 'com.github.mtjsoft:SwiperefreshRecyclerview:1.1.0'
}
```
## GitHub Դ��: [SwiperefreshRecyclerviewԴ��](https://github.com/mtjsoft/SwiperefreshRecyclerview)

## ʹ�� Demo: [SwiperefreshRecyclerviewDemo](https://github.com/mtjsoft/SwiperefreshRecyclerviewDemo)

### ���Բ���:

![���Բ���](http://img.blog.csdn.net/20170624131953706?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjg3NzkwODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### ���񲼾�:

![���񲼾�](http://img.blog.csdn.net/20170624132014859?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjg3NzkwODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### ��ʽ����:

![��ʽ����](http://img.blog.csdn.net/20170624132037025?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjg3NzkwODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 1��activity �̳� HHBaseListRecyclerViewActivity: 
 ��getListDataInThread()�첽��ȡ���ݡ�
 ��setLayoutManagerType()��������ʵ��  ���Բ��֡����񲼾� �� �����֡�

```
/**
     * �첽��ȡ����
     *
     * @param pageIndex ҳ��
     * @param callBack  �첽��ȡ���ݺ�Ļص� 
     * callback.onResponse(list);
     * callback.onFailure(string);
     */
    @Override
    protected void getListDataInThread(int pageIndex, final  NetCallBack<DataModel> callBack) {
    }
    /**
     * ����recyclerview ��adapter
     */
    @Override
    protected HHBaseRecyclerViewAdapter<DataModel> instanceAdapter(List<DataModel> list) {
       //return �Լ���adapter
        return new MyAdapterDemo(getContext(), list);
    }
    /**
     * ����item֮��ľ���
     *
     * @return
     */
    @Override
    protected int setItemDecoration() {
        return 10;
    }

    /**
     * ����ÿҳ��ȡ���ݵĴ�С
     *
     * @return
     */
    @Override
    protected int setPageSize() {
        return 30;
    }

    /**
     * ����LayoutManager���ͣ�Ĭ��2
     * ��
     * 0��LinearLayoutManager ��
     * 1��GridLayoutManager��
     * 2��StaggeredGridLayoutManager
     * ��
     * ����1��2ʱ��setCount��������������������Ĭ��2
     *
     * @return
     */
    @Override
    protected int setLayoutManagerType() {
        return 2;
    }

    /**
     * ����ÿ��������Ĭ��2
     */
    @Override
    protected int setCount() {
        return 2;
    }
```

## 2��Fragment �̳� HHBaseListRecyclerViewFragment ��ʹ�÷����� Activity �̳�HHBaseListRecyclerViewActivity��һ�¡�
**��fragment��ʵ�֣�����Ƕ��������activity��ʹ�á�**


## 3��adapter �̳� HHBaseRecyclerViewAdapter : 
ͨ���������������Ϳ���ʵ��view���ã����ݰ󶨣��򵥸�Ч
```
    /**
     * ����item����
     */
    @Override
    protected int getViewHolderLaoutId() {
        return R.layout.item;//�Լ���item.xml
    }
    /**
     * ������
     */
    @Override
    protected void bindViewHolderData(HHBaseViewHolder holder, int position) {
    //ͨ��holder�õ��ؼ���ͨ��position�õ���Ӧ���ݣ��������ݰ�
    }
```
##4��activity  �� fragment �п��������������µȷ�����

```
    /**
     * �����Ƿ�����ˢ��
     */
    setIsRefresh(Boolean refresh)

    /**
     * �����Ƿ���ظ���
     *
     * @param load_more true��
     */
    setIsLoadMore(Boolean load_more)


    /**
     * ���õײ����ظ����loading���֣�������ʱ��ʹ��Ĭ�ϲ���
     *
     * @param footView
     */
    setFootView(View footView)

    /**
     * ��ȡ�б�����
     */
    getPagerListData()

    /**
     * ��ȡrecyclerview
     */
    getRecyclerView()

    /**
     * ˢ��ҳ������
     */
    onRefresh()
    
    /**
     * ����ҳ��
     *
     * @param pageIndex
     */
    setPageIndex(int pageIndex)

    /**
     * ��ǰҳ��
     */
    getPageIndex()
```
