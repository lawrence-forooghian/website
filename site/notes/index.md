---
---

# Notes

{% assign has_posts = false %}
{% for post in site.posts %}
{% if post.categories contains 'notes' %}
{% assign has_posts = true %}

- [{{ post.title }}]({{ post.url }})
  {% endif %}
  {% endfor %}

{% unless has_posts %}
There’s nothing here yet.
{% endunless %}
