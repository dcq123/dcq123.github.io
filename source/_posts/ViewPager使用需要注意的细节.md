---
title: ViewPager使用需要注意的细节
date: 2016-03-06 10:27:07
tags: Android
---

> ViewPager是Support包中提供的类，使用也是相当广泛了，但是在使用中不同的场景可能需要不同的效果，此时就需要对ViewPager内部做些简单了解，了解其中一些可能变成坑的地方。

#### 默认多加载一页
> ViewPager在直接使用时默认会帮助我们加载下一页的界面及数据，当滚动到中间时，会保留前一个和后一个页面不被销毁，但是有些情况下我们可能不想让ViewPager去加载下一页页面及数据，例如一些新闻APP，大多就是使用ViewPager+Fragment，当用户没有滑动到下一页就自动去加载下一页数据时显然会耗费用户流量，此时就需要修改ViewPage的这种默认加载。

查看ViewPager代码会发现其定义了这个常量：

	private static final int DEFAULT_OFFSCREEN_PAGES = 1;
这个常量就是预加载的页面数量，可以直接根据需要将其设置为0，禁止默认加载。

ps:由于ViewPager类在Support包中是一个独立的类，所以我们完全可以将其拿出来，对其进行定制修改，只需要全部复制，更换个包名即可放到我们自己的项目中使用。这样也就可以随意修改ViewPager的源码了。

#### ViewPager不动态刷新的问题

> ViewPager不能动态刷新，主要是因为其中的PagerAdapter的notifyDataSetChanged()方法会失效。其原因就是当ViewPager绘制完Item之后，ViewPager会将child标记成POSITION_UNCHANGED，这样就不会在notifyDataSetChanged调用时去更新这个Item中的View了，所以界面这个方法是重新PagerAdapter的getItemPosition()方法。


{% codeblock lang:java %}
	@Override
    public int getItemPosition(Object object) {
		// 该方法源码中默认返回POSITION_UNCHANGED，记录为不再改变
        return POSITION_NONE;
    }
{% endcodeblock %}


当我们调用PagerAdapter的notifyDataSetChanged方法之后，系统会去Adapter的getItemPosition方法中遍历所有的child，我们在上面的方法中改写了返回值，全部返回为POSITION_NONE，表示child都没有绘制过，这样ViewPager就会去重绘了。

更加优化一点的代码如下：

{% codeblock lang:java %}
	 @Override
    public void notifyDataSetChanged() {
        mChildCount = getCount();
        super.notifyDataSetChanged();
    }

    @Override
    public int getItemPosition(Object object) {
        // 重写getItemPosition,保证每次获取时都强制重绘UI
        if (mChildCount > 0) {
            mChildCount--;
            return POSITION_NONE;
        }
        return super.getItemPosition(object);
    }
{% endcodeblock %}

增加一个mChildCount的变量用来记录所有Item的数量，避免在数据集无变化时调用notifyDataSetChanged()导致不必要的重绘动作，重而避免增加系统开销，因为重绘的时候，ViewPager会的Destory Item。

更加优化的方法

当我们只需要对ViewPager中的某些元素进行更新时，我们可以在instantiateItem方法调用时，用View.setTag方法加入标志，在需要更新View时，通过findViewWithTag的方法找到对应的View进行更新。

#### 点击按钮切换ViewPager时会看不到切换动画的问题

场景描述：
> 在需要通过两个左右箭头图片点击来实现ViewPager的切换时，往往会看不到想滑动时候的那种动画效果，主要原因是在点击进行切换时，ViewPager的切换速度太快导致基本看不见动画pager就已经切换了。这时就需要对ViewPager的滚动进行控制。ViewPager内部维护了一个Scroller专门负责计算滚动距离，使用它时会根据距离和速度计算滚动持续的时间，而且ViewPager的Scroller在初始化时设置了一个加速的动画插入器(Interpolator)。

在ViewPager的initViewPager()中对Scroller进行了初始化。

	mScroller = new Scroller(context, sInterpolator);

sInterpolator在ViewPager中的实现如下：

{% codeblock lang:java %}
	private static final Interpolator sInterpolator = new Interpolator() {
        public float getInterpolation(float t) {
            --t;
            return t * t * t * t * t + 1.0F;
        }
    };
{% endcodeblock %}

真正进行滚动切换是在smoothScrollTo(int x, int y, int velocity)中：

{% codeblock lang:java %}
	velocity = Math.abs(velocity);
    int duration1;
    if(velocity > 0) {
		// 1、滑动的时候计算出来的滚动时间
        duration1 = 4 * Math.round(1000.0F * Math.abs(distance / (float)velocity));
    } else {
		// 2、不是滑动时计算出来的滚动时间
        float pageWidth = (float)width * this.mAdapter.getPageWidth(this.mCurItem);
        float pageDelta = (float)Math.abs(dx) / (pageWidth + (float)this.mPageMargin);
        duration1 = (int)((pageDelta + 1.0F) * 100.0F);
    }
	// 根据计算出来的时间和默认设置的最大时间常量进行取小者操作
	// 也就表示ViewPager最大的滚动时间上限是600ms
    duration1 = Math.min(duration1, 600);
	// 设置Scroller开始滚动
    this.mScroller.startScroll(sx, sy, dx, dy, duration1);
    ViewCompat.postInvalidateOnAnimation(this);

{% endcodeblock %}

知道以上地方，我们就可以修改ViewPager的源码，来控制点击切换是的滚动时间，使其能正常看到动画效果。

* 修改初始化Scroller时的动画插入器，将其设置为线性加速，这样动画效果才会看的更平滑一些。

		mScroller = new Scroller(context, new LinearInterpolator());

* 修改smoothScrollTo(int x, int y, int velocity)中最后一部分的代码，控制点击滚动时计算出来的时间：

{% codeblock lang:java %}
		velocity = Math.abs(velocity);
	    if (velocity > 0) {
	        duration = 4 * Math.round(1000 * Math.abs(distance / velocity));
	    } else {
			// 点击切换时的滚动时间设置
	        // 此处是直接使用setCurrentItem方法来切换ViewPager,这种情况动画效果不明显，所以在此增加动画的持续时间
	        duration = 400;
	        mScroller.startScroll(sx, sy, dx, dy, duration);
	        ViewCompat.postInvalidateOnAnimation(this);
	        return;
	    }
		// 滑动切换时的时间设置
	    duration = Math.min(duration, MAX_SETTLE_DURATION);
	    // 滑动切换Item时，如果duration大于300ms，则手动设置其为300ms，保证滑动切换迅速
	    duration = duration > 300 ? 300 : duration;
	    mScroller.startScroll(sx, sy, dx, dy, duration);
	    ViewCompat.postInvalidateOnAnimation(this);

{% endcodeblock %}

通过这样就控制了点击翻页的动画显示。
