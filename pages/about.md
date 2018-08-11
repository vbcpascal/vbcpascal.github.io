---
layout: page
title: About
description: vbcpascal的个人博客
keywords: vbcpascal, 码后炮
comments: true
menu: 关于
permalink: /about/
---

能来这里的人都应该是认识我的吧QAQ。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
