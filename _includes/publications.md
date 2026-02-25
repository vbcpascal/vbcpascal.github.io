{% for item in site.data.publications %}
  - {% if item.link %}[{{ item.title }}]({{ item.link }}){%- else -%}{{ item.title }}{%- endif -%}. {{ item.authors }}. *{{ item.conf }} ({{ item.year }})*.
{% endfor %}
