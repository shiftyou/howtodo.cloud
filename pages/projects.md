---
layout: page
title: Project List
permalink: /projects/
---

<ul>
  {% for project in site.data.projects %}
    <li>
      <a href="/project/{{ project.name }}">{{ project.slug }}</a>
      - {{ project.desc }}
    </li>
  {% endfor %}
</ul>
