---
layout: default
---

{% include_relative README.md %}

[GitHub repo](https://github.com/igor-makarov/IsSwiftLikeRust) 
•
[CONTRIBUTING](docs/CONTRIBUTING)

# Features

{% assign features = site.pages | where: 'layout', 'feature' | sort: 'nav_order' %}
{% for feature in features %}
  <h2><a href="{{ feature.url }}">{{ feature.title }}</a></h2>
  {{ feature.excerpt }} 
  [<a href="{{ feature.url }}">read more</a>]
  {% include feature-status.html content=feature %}
{% endfor %}

