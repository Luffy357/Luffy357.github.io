---
title: clickjacking攻击
date: 2019-01-03 21:45:43
tags:
  - Django2.0
category: 学习笔记
---

clickjacking 攻击又称作点击劫持攻击，是一种在网页中将恶意代码等隐藏在看似无害的内容之下，并诱使用户点击的手段。
<!-- more -->

## clickjacking攻击场景
1、如用户收到一封包含一段视频的电子邮件，但其中的播放按钮并不会真正播放视频，而是链入一个购物网站，这样当用户视图播放视频时，实际是被诱骗而进入了一个购物网站。
2、用户在某个网页中，有个按钮A，但是在这个按钮上面浮了一个透明的 iframe 标签，这个 iframe 标签加载了另一个网页，并且他将这个网页的某个按钮和原网页中的按钮A 重合，所以当用户想点击按钮A的时候，其实是点击的通过 iframe 加载的另外一个网页的按钮。

第一种情况是没有办法避免的，而第二种情况是可以避免的。只要设置不允许使用 iframe 被加载到其他网页中，就可以避免这种行为了。可以通过响应头中设置X-Frame-Options 来设置这种操作，X-Frame-Options 可以设置以下三个值：
 + DENY: 不让任何网页使用 iframe 加载我这个页面。
 + SAMEORIGIN: 只允许在相同域名(也就是自己的网站) 下使用 iframe 加载这个页面
 + ALLOW-FROM origin: 允许任何网页通过 iframe 加载这个网页。
在 Django 中，使用中间件 django.middleware.clickjacking.XFrameOptionsMiddleware 可以堵上这个漏洞，这个中间件设置了 X_Frame-Option 为 SAMEORIGIN，也就是只有在自己的网站下才可以使用 iframe 加载这个网页，这样就可以避免其他别有用心的网页去通过 iframe 去加载了。
