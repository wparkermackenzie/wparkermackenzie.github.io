---
layout: default
---
{% comment %}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Well this was the biggest pain in the ass... I turns out the liquid template
and jekyll have no real good way for handling creating, what is essentially,
a table of contents.
This is the best I could do given that I did not want to experiment any more
than the few hours I put into it, others seem as frustrated as I, and any
simple examples found that looked promising just didn't work (boy I was really
happy when I found the sort:0 example then just as disappointed to find it
only works in an older version of liquid)...
Anyway, this will work good enough with a single caviat:
Start all categories with a capital letter...
The magic which lies behind this really wants to only sort categories by 
date and when the sort mechanism is applied to a alphanumeric (words) list it 
applies a sorting order which would have all of the words which start with 
capitals seperate from those which start with a lower case... (e.g. iPhone 
Development would be at the other end of the list from Integrated Development
Environments). Atleast I found natural_sort which should sort those categories
with numbers in a way that may make sense...-Parker
PS Cannot use variables in hash lookups argh...
  References and Thanks to the following
  - https://gist.github.com/Phlow/a0e3fa686eb259fe7f76
  - https://github.com/Shopify/liquid/wiki/Liquid-for-Designers
  - https://blog.lanyonm.org/articles/2013/11/21/alphabetize-jekyll-page-tags-pure-liquid.html
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{% endcomment %}

{% assign categoriesSorted=site.categories|natural_sort %}
{% for category in categoriesSorted %}
# [](#header-1){{ category[0]|capitalize }}
  {% assign postsSorted=category[1]|sort:'title' %}
  {% for post in postsSorted %}
* [ {{post.title}} ]({{ post.url }})
  {% endfor %}
{% endfor %}
