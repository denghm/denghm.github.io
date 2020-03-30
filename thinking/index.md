---
layout: archive
title: 信息，在于你从多大的不确定性中做出了选择
---






{% if site.categories.thinking.size %}
共有 {{ site.categories.thinking.size }} 篇感悟类文章
		{% else %}
暂无感悟类文章
		{% endif %}

<div class="tiles">
{% for post in site.categories.thinking %}
	{% include post-list-cn.html %}
{% endfor %}
</div><!-- /.tiles -->