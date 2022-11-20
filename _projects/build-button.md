---
title: 'Build button'
permalink: /projects/build-button/
---
A bluetooth button with USB charging for running tests

<ul>
  {% assign posts = site.posts | where:"category","build button" %}
  {% for post in posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
