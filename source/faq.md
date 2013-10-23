---
layout: page
title: FAQs
---

{% for post in site.posts %}
{% if post.categories contains "faq" %}
<!--{{ post.order }}-->
<div class="span12">
<article>
{% include archive_post.html %}
</article>
</div>
{% endif %}
{% endfor %}
