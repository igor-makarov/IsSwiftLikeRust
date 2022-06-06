---
layout: default
---

Swift and Rust are modern programming languages that are designed to be type-safe, performant, and portable.  
And yet the two are rarely compared or even mentioned in the same context.

{% assign features = site.pages | where: 'layout', 'feature' %}
{% for feature in features %}
  <h3><a href="{{ feature.url }}">{{ feature.title }}</a></h3>
  {{ feature.excerpt }}
  [<a href="{{ feature.url }}">more info</a>]
  {% include feature-status.html content=feature %}
{% endfor %}

