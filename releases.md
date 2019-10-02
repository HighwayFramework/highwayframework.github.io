---
layout: page
title: Releases
---

{% for post in site.posts %}
{% if post.categories contains "release" %}
<!--{{ post.order }}-->
<div class="span12">
<article>
{% include archive_post.html %}
</article>
</div>
{% endif %}
{% endfor %}
