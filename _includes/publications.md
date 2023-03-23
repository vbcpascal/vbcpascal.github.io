{% for item in site.data.publications %}
  - {{ item.authors }} ({{ item.year }}). {{ item.title }}. In {{ item.conf }}.
{% endfor %}
