---
layout: projects
title: Projects
permalink: /projects/
---

작은 프로젝트를 담고 있습니다.
<ul>
  {% for project in site.projects %}
    <li>
      <a href="{{ project.url }}">{{ project.title }}</a>
      - {{ project.headline }}
    </li>
  {% endfor %}
</ul>
