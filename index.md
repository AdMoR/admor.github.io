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

<div></div>

<h2>Interactive experiments</h2>
<li>
<a href="/test.html"> De Jong curves explorer </a> 
</li>
<li>
<a href="/ancient_scribble.html"> Experiment with chaikin curves </a>
</li>