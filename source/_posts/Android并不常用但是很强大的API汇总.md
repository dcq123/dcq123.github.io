
---
title: Android并不常用但是很强大的API汇总
date: 2016-03-06 10:27:07
tags: Android
---

> 本文主要用来总结一些开发中不常用，或者并不为人所知的一些强大API。对这些这些Cool的API进行汇总，随时待续添加内容。

#### startActivities()方法

> 其实我们绝大多数的开发者可能是没有用过这个方法的，根据我个人理解，用的到场景并不多。这个方法最直接的理解就是使用intent开启多个Activity，我在Google的关于Activity.startActivities()文档说明中，并没有获取到除了StartActivity之外更多的信息。于是我继续扒源码，果然在Context和ContextCompat 下找到了更加详细的说明和解释。

Context，以及 documentation of ContextCompat 

	* Launch multiple new activities.  This is generally the same as calling
	* {@link #startActivity(Intent)} for the first Intent in the array,
	* that activity during its creation calling {@link #startActivity(Intent)}
	* for the second entry, etc.  Note that unlike that approach, generally
	* none of the activities except the last in the array will be created
	* at this point, but rather will be created when the user first visits
	* them (due to pressing back from the activity on top).

从上面两处文档解释我们可以理解到的，startActiviyies 会创建一个新的Task Stack，任务栈里Activity的位置基于我们传入的Intent数组。当我们按返回键时，就会将栈顶的Activity移除。。其实这也是framework管理用户开启的activity的方式。这里结合Activity的启动模式来理解，就简单多了。

至于应用场景，我目前能想到的就是点击通知栏来开启应用的Activities、APP启动图点击后的跳转逻辑等，而开启哪些Activity以及相应顺序，我们就可以用到这个方法了。按照顺序最后一个Activity展示在栈顶，当按返回键时，一级一级返回到最前面的Activity。

#### ArgbEvaluator

> 这个类提供了方便的颜色渐变设置，据一个起始颜色值和一个结束颜色值以及一个偏移量生成一个新的颜色，分分钟实现类似于微信底部栏滑动颜色渐变。
{% codeblock lang:java %}
	ArgbEvaluator.evaluate(float fraction, Object startValue, Object endValue);
{% endcodeblock %}

fraction的值需要在0-1之间，否则达不到想要的效果；

这里提供另一个颜色渐变的版本，来之官方示例：

{% codeblock lang:java %}
	    **
	    * Blend {@code color1} and {@code color2} using the given ratio.
	    *
	    * @param ratio of which to blend. 1.0 will return {@code color1}, 0.5 will give an even blend,
	    *              0.0 will return {@code color2}.
	    */
	   private static int blendColors(int color1, int color2, float ratio) {
	       final float inverseRation = 1f - ratio;
	       float r = (Color.red(color1) * ratio) + (Color.red(color2) * inverseRation);
	       float g = (Color.green(color1) * ratio) + (Color.green(color2) * inverseRation);
	       float b = (Color.blue(color1) * ratio) + (Color.blue(color2) * inverseRation);
	       return Color.rgb((int) r, (int) g, (int) b);
	   }

{% endcodeblock %}

#### Formatter.formatFileSize()

> 这个方法会格式化数据的大小，根据输入的字节大小，返回 B KB MB GB 等等（最大支持到 PB）。当然要注意的是输入的最大值是 Long.MAX_VALUE.注意： 这个类在 android.text.format 包下。

#### TypedValue.applyDimension()

> 这个方法我们可以用来对sp dp 和 px 之间的单位转换

查看源码：

{% codeblock lang:java %}
	/**
	    * Converts an unpacked complex data value holding a dimension to its final floating point value. The two parameters <var>unit</var> and <var>value</var>
	    * are as in {@link #TYPE_DIMENSION}.
	    *  
	    * @param unit The unit to convert from.
	    * @param value The value to apply the unit to.
	    * @param metrics Current display metrics to use in the conversion -- 
	    *                supplies display density and scaling information.
	    * 
	    * @return The complex floating point value multiplied by the appropriate 
	    * metrics depending on its unit. 
	    */
	   public static float applyDimension(int unit, float value,
	                                      DisplayMetrics metrics)
	   {
	       switch (unit) {
	       case COMPLEX_UNIT_PX:
	           return value;
	       case COMPLEX_UNIT_DIP:
	           return value * metrics.density;
	       case COMPLEX_UNIT_SP:
	           return value * metrics.scaledDensity;
	       case COMPLEX_UNIT_PT:
	           return value * metrics.xdpi * (1.0f/72);
	       case COMPLEX_UNIT_IN:
	           return value * metrics.xdpi;
	       case COMPLEX_UNIT_MM:
	           return value * metrics.xdpi * (1.0f/25.4f);
	       }
	       return 0;
	   }

{% endcodeblock %}

#### ThumbnailUtils
>这个类主要是用来处理缩略图相关的，有过这方面需求的，应该是用过这个类的。而且这个类只有3个静态方法，很简单。至于缩略图是怎么生成的 以及源码的实现，有兴趣的可以去看下。


{% codeblock extractThumbnail (source, width, height) lang:java %}
	   /** 
 		*  
 		* 创建一个指定大小的缩略图 
 		* @param source 源文件(Bitmap类型) 
		* @param width  压缩成的宽度 
 		* @param height 压缩成的高度 
		*/  
		ThumbnailUtils.extractThumbnail(source, width, height);

{% endcodeblock %}

{% codeblock extractThumbnail(source, width, height, options) lang:java %}

		/** 
		 * 创建一个指定大小居中的缩略图 
		 *  
		 * @param source 源文件(Bitmap类型) 
		 * @param width  输出缩略图的宽度 
		 * @param height 输出缩略图的高度 
		 * @param options 如果options定义为OPTIONS_RECYCLE_INPUT,则回收@param source这个资源文件 
		 * (除非缩略图等于@param source) 
		 *  
		 */  
		ThumbnailUtils.extractThumbnail(source, width, height, options);

{% endcodeblock %}

{% codeblock createVideoThumbnail(filePath, kind) lang:java %}

		/** 
		 * 创建一张视频的缩略图 
		 * 如果视频已损坏或者格式不支持可能返回null 
		 *  
		 * @param filePath 视频文件路径  如：/sdcard/android.3gp 
		 * @param kind kind可以为MINI_KIND或MICRO_KIND 
		 *  
		 */  
		ThumbnailUtils.createVideoThumbnail(filePath, kind);
{% endcodeblock %}
