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
	On generative art
	<a href=""> De Jong curves explorer </a>
</ul>
