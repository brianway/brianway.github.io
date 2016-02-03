---
layout: page
title: Archive
permalink: /archive/
icon: glyphicon-folder-open
---

* content
{:toc}

{% for post in site.posts  %}
    {% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
    {% capture this_month %}{{ post.date | date: "%m" }}{% endcapture %}
    {% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
    {% capture next_month %}{{ post.previous.date | date: "%m" }}{% endcapture %}

    {% if forloop.first %}
	
## {{this_year}}年-{{this_month}}月
--------------
    {% endif %}

- {{ post.date | date: "%Y年-%m月-%d日" }} >> [{{ post.title }}]({{ post.url }})

    {% if forloop.last %}
	
    {% else %}
      {% if this_year != next_year %}
## {{next_year}}年-{{next_month}}月
--------------
      {% else %}    
        {% if this_month != next_month %}
## {{next_year}}年-{{next_month}}月
--------------
        {% endif %}
      {% endif %}
    {% endif %}

{% endfor %} 