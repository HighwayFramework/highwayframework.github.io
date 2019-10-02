---
layout: project-data
project: data
---

<h1>Features</h1>
{% assign posts = site.posts | sort:"order" %}
{% for post in posts %}
{% if post.categories contains "feature" and post.categories contains page.project %}
<!--{{ post.order }}-->
<div class="span12">
<article>
{% include feature_post.html %}
</article>
</div>
{% endif %}
{% endfor %}
