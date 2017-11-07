---
layout: post
title:  "RecyclerView实现添加头部和底部"
subtitle:  "RecyclerView、头部、底部"
date:  2017-05-04 15:00
author:  "Mtj"
tags:
     Android
     Recyclerview
     头部、底部
     
---
# 1、主要用到下面四个方法。

![这里写图片描述](http://img.blog.csdn.net/20170504152947129?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjg3NzkwODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170504153018176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjg3NzkwODM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
# 2、方法开源于 鸿洋_ 的这篇文章 [ Android 优雅的为RecyclerView添加HeaderView和FooterView ](http://blog.csdn.net/lmj623565791/article/details/51854533)

#  大神是通过类似装饰者模式，去设计一个类，在不改变原有的Adapter基础上去增强其功能，使其支持addHeaderView和addFooterView。

# 3、而此篇文章是直接对RecyclerView.Adapter动手，使其支持addHeaderView和addFooterView，removeHeaderView和removeFooterView。

# 4、原理大神已给出，直接代码。直接继承此adapter即可。
```
import java.util.List;

import android.content.Context;
import android.support.v4.util.SparseArrayCompat;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.StaggeredGridLayoutManager;
import android.view.View;
import android.view.ViewGroup;

import com.huahan.readerviewpager.adapter.ViewHolder;

/**
 * Created by android.mtj on 2017/4/1.
 */

public abstract class BaseRecyclerViewAdapter<T> extends
		RecyclerView.Adapter<ViewHolder> {

	private static final int BASE_ITEM_TYPE_HEADER = 100000;
	private static final int BASE_ITEM_TYPE_FOOTER = 200000;
	private SparseArrayCompat<View> mHeaderViews = new SparseArrayCompat<>();
	private SparseArrayCompat<View> mFootViews = new SparseArrayCompat<>();
	private Context context;
	private List<T> list;

	public BaseRecyclerViewAdapter(Context context, List<T> list) {
		this.context = context;
		this.list = list;
	}

	@Override
	public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
		ViewHolder holder = null;
		if (mHeaderViews.get(viewType) != null) {
			holder = ViewHolder.createViewHolder(context,
					mHeaderViews.get(viewType));
		} else if (mFootViews.get(viewType) != null) {
			holder = ViewHolder.createViewHolder(context,
					mFootViews.get(viewType));
		} else {
			holder = ViewHolder.createViewHolder(context, parent,
					getViewHolderLaoutId());
		}
		return holder;
	}

	/**
	 * 获取ViewHolder layoutID
	 * 
	 * @return
	 */
	protected abstract int getViewHolderLaoutId();

	/**
	 * 绑定ViewHolder
	 */
	@Override
	public void onBindViewHolder(ViewHolder holder, int position) {
		if (!(isHeaderViewPos(position) || isFooterViewPos(position))) {
			bindViewHolderData(holder, position);
		}
	}

	/**
	 * 子类实现绑定ViewHolder
	 * 
	 * @param holder
	 * @param position
	 */
	protected abstract void bindViewHolderData(ViewHolder holder, int position);

	/**
	 * 获取数据
	 * 
	 * @return
	 */
	public List<T> getListData() {
		return list;
	}

	/**
	 * 获取总条目
	 */
	@Override
	public int getItemCount() {
		return getHeadersCount() + getRealItemCount() + getFootersCount();
	}

	/**
	 * 判断是否是头部
	 * 
	 * @param position
	 * @return
	 */
	private boolean isHeaderViewPos(int position) {
		return position < getHeadersCount();
	}

	/**
	 * 判断是否是底部
	 * 
	 * @param position
	 * @return
	 */
	private boolean isFooterViewPos(int position) {
		return position >= getHeadersCount() + getRealItemCount();
	}

	/**
	 * 添加头部
	 * 
	 * @param view
	 */
	public void addHeaderView(View view) {
		mHeaderViews.put(mHeaderViews.size() + BASE_ITEM_TYPE_HEADER, view);
	}

	/**
	 * 添加底部
	 * 
	 * @param view
	 */
	public void addFootView(View viewid) {
		mFootViews.put(mFootViews.size() + BASE_ITEM_TYPE_FOOTER, viewid);
	}

	/**
	 * 获取数据条目
	 * 
	 * @return
	 */
	public int getRealItemCount() {
		return list.size();
	}

	/**
	 * 获取头部条目
	 * 
	 * @return
	 */
	public int getHeadersCount() {
		return mHeaderViews.size();
	}

	/**
	 * 获取底部条目
	 * 
	 * @return
	 */
	public int getFootersCount() {
		return mFootViews.size();
	}

	/**
	 * 移除第 i 个头部（0开始）
	 * 
	 * @param i
	 */
	public void removeHeaderView(int i) {
		if (mHeaderViews.get(BASE_ITEM_TYPE_HEADER + i) != null) {
			mHeaderViews.remove(BASE_ITEM_TYPE_HEADER + i);
		}
	}

	/**
	 * 移除第 i 个底部（0开始）
	 * 
	 * @param i
	 */
	public void removeFooterView(int i) {
		if (mFootViews.get(BASE_ITEM_TYPE_FOOTER + i) != null) {
			mFootViews.remove(BASE_ITEM_TYPE_FOOTER + i);
		}
	}

	@Override
	public int getItemViewType(int position) {
		if (isHeaderViewPos(position)) {
			return mHeaderViews.keyAt(position);
		} else if (isFooterViewPos(position)) {
			return mFootViews.keyAt(position - getHeadersCount()
					- getRealItemCount());
		}
		return position - getHeadersCount();
	}

	/**
	 * GridLayoutManager 时，头部和底部都要占据整行
	 *
	 * @param recyclerView
	 */
	@Override
	public void onAttachedToRecyclerView(RecyclerView recyclerView) {
		super.onAttachedToRecyclerView(recyclerView);
		if (recyclerView == null) {
			return;
		}
		final RecyclerView.LayoutManager layoutManager = recyclerView
				.getLayoutManager();
		if (layoutManager instanceof GridLayoutManager) {
			((GridLayoutManager) layoutManager)
					.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
						@Override
						public int getSpanSize(int position) {
							return (isHeaderViewPos(position) || isFooterViewPos(position)) ? ((GridLayoutManager) layoutManager)
									.getSpanCount() : 1;
						}
					});
		}
	}

	/**
	 * StaggeredGridLayoutManager 时，头部和底部都要占据整行
	 * 
	 * @param holder
	 */
	@Override
	public void onViewAttachedToWindow(ViewHolder holder) {
		super.onViewAttachedToWindow(holder);
		int position = holder.getLayoutPosition();
		if (isHeaderViewPos(position) || isFooterViewPos(position)) {
			ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
			if (lp != null
					&& lp instanceof StaggeredGridLayoutManager.LayoutParams) {
				StaggeredGridLayoutManager.LayoutParams p = (StaggeredGridLayoutManager.LayoutParams) lp;
				p.setFullSpan(true);
			}
		}
	}
}
```
# 5、文中用到的ViewHolder的封装

```
import android.annotation.SuppressLint;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Paint;
import android.graphics.Typeface;
import android.graphics.drawable.Drawable;
import android.os.Build;
import android.support.v7.widget.RecyclerView;
import android.text.util.Linkify;
import android.util.SparseArray;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.view.animation.AlphaAnimation;
import android.widget.Checkable;
import android.widget.ImageView;
import android.widget.ProgressBar;
import android.widget.RatingBar;
import android.widget.TextView;

public class ViewHolder extends RecyclerView.ViewHolder {
	private SparseArray<View> mViews;
	private View mConvertView;
	private Context mContext;

	public ViewHolder(Context context, View itemView) {
		super(itemView);
		mContext = context;
		mConvertView = itemView;
		mViews = new SparseArray<View>();
	}

	public static ViewHolder createViewHolder(Context context, View itemView) {
		ViewHolder holder = new ViewHolder(context, itemView);
		return holder;
	}

	public static ViewHolder createViewHolder(Context context,
			ViewGroup parent, int layoutId) {
		View itemView = LayoutInflater.from(context).inflate(layoutId, parent,
				false);
		ViewHolder holder = new ViewHolder(context, itemView);
		return holder;
	}

	/**
	 * 通过viewId获取控件
	 *
	 * @param viewId
	 * @return
	 */
	public <T extends View> T getView(int viewId) {
		View view = mViews.get(viewId);
		if (view == null) {
			view = mConvertView.findViewById(viewId);
			mViews.put(viewId, view);
		}
		return (T) view;
	}

	public View getConvertView() {
		return mConvertView;
	}

	/**** 以下为辅助方法 *****/
	/**
	 * 获取控件
	 */
	public ImageView getImageView(int viewId) {
		ImageView iv = getView(viewId);
		return iv;
	}
	
	public TextView getTextView(int viewId) {
		TextView tv = getView(viewId);
		return tv;
	}

	/**
	 * 设置TextView的值
	 *
	 * @param viewId
	 * @param text
	 * @return
	 */
	public ViewHolder setText(int viewId, String text) {
		TextView tv = getView(viewId);
		tv.setText(text);
		return this;
	}

	public ViewHolder setImageResource(int viewId, int resId) {
		ImageView view = getView(viewId);
		view.setImageResource(resId);
		return this;
	}

	public ViewHolder setImageBitmap(int viewId, Bitmap bitmap) {
		ImageView view = getView(viewId);
		view.setImageBitmap(bitmap);
		return this;
	}

	public ViewHolder setImageDrawable(int viewId, Drawable drawable) {
		ImageView view = getView(viewId);
		view.setImageDrawable(drawable);
		return this;
	}

	public ViewHolder setBackgroundColor(int viewId, int color) {
		View view = getView(viewId);
		view.setBackgroundColor(color);
		return this;
	}

	public ViewHolder setBackgroundRes(int viewId, int backgroundRes) {
		View view = getView(viewId);
		view.setBackgroundResource(backgroundRes);
		return this;
	}

	public ViewHolder setTextColor(int viewId, int textColor) {
		TextView view = getView(viewId);
		view.setTextColor(textColor);
		return this;
	}

	public ViewHolder setTextColorRes(int viewId, int textColorRes) {
		TextView view = getView(viewId);
		view.setTextColor(mContext.getResources().getColor(textColorRes));
		return this;
	}

	@SuppressLint("NewApi")
	public ViewHolder setAlpha(int viewId, float value) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
			getView(viewId).setAlpha(value);
		} else {
			// Pre-honeycomb hack to set Alpha value
			AlphaAnimation alpha = new AlphaAnimation(value, value);
			alpha.setDuration(0);
			alpha.setFillAfter(true);
			getView(viewId).startAnimation(alpha);
		}
		return this;
	}

	public ViewHolder setVisible(int viewId, boolean visible) {
		View view = getView(viewId);
		view.setVisibility(visible ? View.VISIBLE : View.GONE);
		return this;
	}

	public ViewHolder linkify(int viewId) {
		TextView view = getView(viewId);
		Linkify.addLinks(view, Linkify.ALL);
		return this;
	}

	public ViewHolder setTypeface(Typeface typeface, int... viewIds) {
		for (int viewId : viewIds) {
			TextView view = getView(viewId);
			view.setTypeface(typeface);
			view.setPaintFlags(view.getPaintFlags() | Paint.SUBPIXEL_TEXT_FLAG);
		}
		return this;
	}

	public ViewHolder setProgress(int viewId, int progress) {
		ProgressBar view = getView(viewId);
		view.setProgress(progress);
		return this;
	}

	public ViewHolder setProgress(int viewId, int progress, int max) {
		ProgressBar view = getView(viewId);
		view.setMax(max);
		view.setProgress(progress);
		return this;
	}

	public ViewHolder setMax(int viewId, int max) {
		ProgressBar view = getView(viewId);
		view.setMax(max);
		return this;
	}

	public ViewHolder setRating(int viewId, float rating) {
		RatingBar view = getView(viewId);
		view.setRating(rating);
		return this;
	}

	public ViewHolder setRating(int viewId, float rating, int max) {
		RatingBar view = getView(viewId);
		view.setMax(max);
		view.setRating(rating);
		return this;
	}

	public ViewHolder setTag(int viewId, Object tag) {
		View view = getView(viewId);
		view.setTag(tag);
		return this;
	}

	public ViewHolder setTag(int viewId, int key, Object tag) {
		View view = getView(viewId);
		view.setTag(key, tag);
		return this;
	}

	public ViewHolder setChecked(int viewId, boolean checked) {
		Checkable view = (Checkable) getView(viewId);
		view.setChecked(checked);
		return this;
	}

	/**
	 * 关于事件的
	 */
	public ViewHolder setOnClickListener(int viewId,
			View.OnClickListener listener) {
		View view = getView(viewId);
		view.setOnClickListener(listener);
		return this;
	}

	public ViewHolder setOnTouchListener(int viewId,
			View.OnTouchListener listener) {
		View view = getView(viewId);
		view.setOnTouchListener(listener);
		return this;
	}

	public ViewHolder setOnLongClickListener(int viewId,
			View.OnLongClickListener listener) {
		View view = getView(viewId);
		view.setOnLongClickListener(listener);
		return this;
	}

}
```
