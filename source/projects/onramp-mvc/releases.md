---
layout: project-onramp-mvc
project: onramp-mvc
---

<h1>Releases</h1>
{% for post in site.posts %}
{% if post.categories contains "release" and post.categories contains page.project %}
<!--{{ post.order }}-->
<div class="span12">
<article>
{% include feature_post.html %}
</article>
</div>
</div>
{% endif %}
{% endfor %}