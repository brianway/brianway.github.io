---
layout: page
title: Links
permalink: /links/
icon: glyphicon-paperclip
---

* content
{:toc}

## 友情链接

-----

{% for contact in site.data.links %}
	{% assign i = contact[1] %}
### {{ i.message }}
---
		{% for link in i.items %}
- [{{ link.text }}]({{ link.url }}) 
		{% endfor %}

---
{% endfor %}

