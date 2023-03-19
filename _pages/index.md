---
layout: page
title: Home
id: home
permalink: /
---

# Welcome! 🌱

<p style="padding: 3em 1em; background: #092124; border-radius: 4px;">
  Take a look at <span style="font-weight: bold">[[Your first note]]</span> to get started on your exploration.
</p>

This digital garden template is free, open-source, and [available on GitHub here](https://github.com/maximevaillancourt/digital-garden-jekyll-template).

The easiest way to get started is to read this [step-by-step guide explaining how to set this up from scratch](https://maximevaillancourt.com/blog/setting-up-your-own-digital-garden-with-jekyll).

<strong>Recently updated notes</strong>

<ul>
  {% assign recent_notes = site.notes | sort: "last_modified_at_timestamp" | reverse %}
  {% for note in recent_notes limit: 5 %}
    <li>
      {{ note.last_modified_at | date: "%Y-%m-%d" }} — <a class="internal-link" href="{{ note.url }}">{{ note.title }}</a>
    </li>
  {% endfor %}
</ul>

<hr />

**Don't know where to start?** Here are a few tags to get started.

{% assign recommended_tags = "web,meta" | split: ',' %}
{% for tag in recommended_tags %}
##### Recent <span class="tag">{{ tag }}</span> pages
  <ul>
    {% assign tag_notes = site.notes | where: 'tags', tag | sort: "last_modified_at_timestamp" | reverse %}
    {% for note in tag_notes limit: 5 %}
      <li>
        {{ note.last_modified_at | date: "%Y-%m-%d" }} — <a class="internal-link" href="{{ note.url }}">{{ note.title }}</a>
      </li>
    {% endfor %}
  </ul>
{% endfor %}


<style>
  .wrapper {
    max-width: 46em;
  }
</style>
