---
layout: default
---

Swift and Rust are modern programming languages that are designed to be type-safe, performant, and portable.  
And yet the two are rarely compared or even mentioned in the same context.

{% for page in site.pages %}
  {% if page.kind == 'feature' %}
### {{ page.title }}
{{ page.content }}
{% include feature-status.html content=page %}

  {% endif %}
{% endfor %}

