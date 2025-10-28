---
layout: home
title: Welcome
---

![AWS Logo](/assets/images/icons8-aws-logo-100.png)

**AWS Zero to Architect** – hands-on blog series.

{% for post in site.posts limit:5 %}
- **[{{ post.date | date: "%b %d, %Y" }}] {{ post.title }}** – {{ post.description }}
{% endfor %}

[View all posts →](/posts)
