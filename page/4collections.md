---
layout: page
title: Links
permalink: /collection/
icon: bookmark
---

* content
{:toc}



{% for contact in site.data.links %}
	{% assign i = contact[1] %}
### {{ i.message }}
---
		{% for link in i.items %}
- [{{ link.text }}]({{ link.url }}) 
		{% endfor %}

---
{% endfor %}



## Comments

{% include comments.html %}

