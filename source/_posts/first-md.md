---
title: 测试hexo框架
date: 2016-05-05 17:34:17
categories: 测试类
tags: Android
---

### 测试hexo框架

{% blockquote %}
这事一个测试沙发看哈教科书地方
{% endblockquote %}

{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

代码部分：
{% codeblock 布局文件 lang:xml %}
	<LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_above="@+id/logout"
        android:layout_below="@id/menu_header"
        android:layout_marginTop="@dimen/pt_12"
        android:clickable="true"
        android:orientation="vertical">

        <include
            android:id="@+id/client_code"
            layout="@layout/menu_item" />

        <include
            android:id="@+id/client_name"
            layout="@layout/menu_item" />

        <include
            android:id="@+id/reset_password"
            layout="@layout/menu_item"
            android:layout_width="match_parent"
            android:layout_height="@dimen/pt_45"
            android:layout_marginTop="@dimen/pt_12" />

        <include
            android:id="@+id/about"
            layout="@layout/menu_item" />

    </LinearLayout>
{% endcodeblock %}
##### 分了标题

1. 第一条
2. 第二条
3. 第三条

#### 侧四测试啊还是地方艰苦
