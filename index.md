---
layout: default
---


<h2> Some of my posts </h2>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      
    </li>
  {% endfor %}
</ul>
