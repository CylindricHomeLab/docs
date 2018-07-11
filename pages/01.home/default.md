---
title: Home
taxonomy:
    category:
        - Home
    tag:
        - home
content:
    items: '@self.children'
body_classes: 'title-center title-h1h2'
---

Hello

{% for category,value in page.header.taxonomy.category %}
{{ category }}{{ value }}

{% endfor %}