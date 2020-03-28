---
layout: default
---

<h2> Some of my posts </h2>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {{ post.description }}
 
  {% endfor %}
</ul>

<ul>
	<h4>Interactive experiments</h4>
	<a href="/test.html"> De Jong curves explorer </a>
</ul>
