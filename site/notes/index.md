---
title: Notes
noindex: true
---

# Notes

These are intended just as notes to myself. But I'm hoping that making them public will:

- stop me from dumping notes into files on my computer that I promise myself I’ll do something useful with one day
- make me feel comfortable showing people low-quality, unfinished things
- maybe be a handy thing to send someone a link to once in a while when I need to explain something
- give me a starting point when people ask me what I’ve been doing / thinking about recently, which I am terrible at answering

## Contents

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
