---
title: "임시"
layout: archive
permalink: categories/temp
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.["temp"] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}