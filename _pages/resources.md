---
layout: archive 
permalink: /resources/
title: "Resources"
author_profile: true
---
{% include base_path %}

{% for post in site.resources reversed %}
  {% include archive-single.html %}
{% endfor %}