---
layout: page
title: About
description: 妄图改变世界
keywords: Rogers Cai, 蔡鎔玖
comments: true
menu: 关于
permalink: /about/
---

我是玖糕老师，真的艺术人，假的程序员。

仰慕「优雅编码的艺术」。

坚信熟能生巧，努力改变人生。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'blog.nbcairongjiu.cn' %}
{% endif %}
</ul>

## Skill Keywords

{% for skill in site.data.skills %}

### {{ skill.name }}

<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
