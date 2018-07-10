---
title: Home
taxonomy:
    category:
        - Home
    tag:
        - home
body_classes: 'title-center title-h1h2'
process:
    markdown: true
    twig: true
---

Hello

{% for category,value in page.header.taxonomy.category %}
{{ category }}{{ value }}

{% endfor %}