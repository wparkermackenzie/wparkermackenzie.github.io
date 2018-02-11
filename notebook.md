---
layout: default
---
{% assign categories_sorted = site.categories | sort %}
{% for catagory in categories_sorted  %}
# [](#header-1){{ category[0]|capitalize }}
  {% for post in category[1] | sort:'title' %}
  [ {{post.title}} ]({{ post.url }})
  {% endfor %}
{% endfor %}

{% comment %}
 for category in site.categories  
  Thanks to the following
  - https://gist.github.com/Phlow/a0e3fa686eb259fe7f76
  - https://github.com/Shopify/liquid/wiki/Liquid-for-Designers
{% endcomment %}
