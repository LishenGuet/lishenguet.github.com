---
layout: page
title: Welcome
tagline: 技术，技术，还是技术！
---
{% include JB/setup %}

## 我是一个
编程菜鸟 | 产品爱好者 | 学习狂

## 我喜欢
读书 | 中文经典老歌 | 旅游 | 摄影 | 健身，球类运动

## Follow me @
[微博](http://weibo.com/eric3300) | [人人](http://www.renren.com/248642647/profile) | [我的生活博客](http://lishen.sinaapp.com)

## 最近发表的文章:
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>



