---
---
{% for doc in site.posts %}
doc.name: {{ doc.name }} | doc.path: {{ doc.path }}
{% endfor %}
