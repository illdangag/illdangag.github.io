---
title: "Nginx"
layout: archive
permalink: categories/nginx
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.Nginx %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}