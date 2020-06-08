---
layout: page
title: Leetcode
description: 人越学越觉得自己无知
keywords: 维基, Wiki
comments: false
menu: 维基
permalink: /leetcode/
---

> 记多少命令和快捷键会让脑袋爆炸呢？

<ul class="listing">
{% for wiki in site.leetcode %}
{% if wiki.title != "Wiki Template" and wiki.topmost == true %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}"><span class="top-most-flag">[置顶]</span>{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" and wiki.topmost != true %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>