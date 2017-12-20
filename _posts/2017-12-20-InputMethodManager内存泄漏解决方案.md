---
title:      Android Memory Leaks InputMethodManager Solved
subtitle:   主Acitivity在finish后并没有被回收，因为它被InputMethodManager间接引用
date:       2017-12-20
author:     小菜
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Android
---

> **InputMethodManager导致的内存泄漏**

我之前做的一个开源项目<a href="https://github.com/xiaokun19931126/Meizi">Meizi</a>,发现了这个问题。我双击退出程序后还是会有这个内存泄漏问题。这个问题主要是因为在Activity中使用了ViewPager+Fragment+FragmentStatePagerAdapter+RecyclerView，导致Activity close之后还被TreeObserver或者其他东西引用着。

解决方法如下：创建一个透明的空的DummyActivity，然后在HomeActivity finish之前跳转到DummyActivity上。



```
public class DummyActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                finish();
            }
        }, 500);
    }
}
```





