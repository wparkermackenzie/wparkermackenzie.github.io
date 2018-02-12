---
layout: default
---
### Welcome. 
This site has been published with three main objectives in mind. First, it is a place to store and share technical notes. Second, it is a place to give back in the hope that I may be as much help, as others have been for me. Third, this site allows you to get to know who I am as an engineer; whether it is to collaborate on an open source project, an aid in evaluating my skillset for an upcoming private project, or simply read the ramblings of a like minded individual.

In the above header, I have included numerous ways I may be contacted; whether it is social media or an e-mail, I look forward to hearing from you. More information about me may be found in my resume.

---

### Recent Posts

{% for post in site.posts %}
  {% if forloop.index == 3 %}
    {% break %}
  {% endif %}
  [ {{post.date | date_to_string}}: {{ post.title }}]({{ post.url }})
  {{ post.excerpt }}
{% endfor %}


