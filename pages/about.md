---
layout: page
title: About
description:
keywords: Bo Wen, 问博
comments: true
menu: 关于
permalink: /about/
---

I'm Bo Wen。

Currenly living in Hangzhou, China.

## Contact(联系)

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>


## Interests(兴趣)

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
