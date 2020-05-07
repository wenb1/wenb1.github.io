---
layout: page
title: About
description: Talk is cheap, show me your code
keywords: Bo Wen, 问博
comments: true
menu: 关于
permalink: /about/
---

I'm Bo Wen, future master student in computing and software.
Currenly living in Hamilton, Ontario, Canada.
Enthusiastic about computer science.

## Contact(联系)

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>


## Skill Sets(技术栈)

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
