---
layout: default
---

{% include_relative README.md %}

# Features

{% assign features = site.pages | where: 'layout', 'feature' %}
{% for feature in features %}
  <h2><a href="{{ feature.url }}">{{ feature.title }}</a></h2>
  {{ feature.excerpt }}
  [<a href="{{ feature.url }}">more info</a>]
  {% include feature-status.html content=feature %}
{% endfor %}

