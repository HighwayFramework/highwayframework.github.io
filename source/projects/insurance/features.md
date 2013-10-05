---
layout: project-insurance
project: insurance
---

<h1>Features</h1>
{% sorted_for post in site.posts sort_by:order %}
{% if post.categories contains "feature" and post.categories contains page.project %}
<!--{{ post.order }}-->
<div class="span12">
<article>
{% include feature_post.html %}
</article>
</div>
</div>
{% endif %}
{% endsorted_for %}
