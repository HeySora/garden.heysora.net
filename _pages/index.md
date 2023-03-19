---
layout: page
title: Home
id: home
permalink: /
---

# Hey! 🦊

This garden is an early Work in Progress! Feel free to take a look at **[[garden-plans|my current plans]]**.

<hr />

{% include notes_graph.html %}

<hr />

**Recently updated notes**

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
