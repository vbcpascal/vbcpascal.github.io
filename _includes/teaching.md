{% for item in site.data.teaching %}
- *{{ item.role }}.*
  {% if item.site %}[{{ item.course }}]({{item.site}})
  {%- else -%}{{ item.course }}
  {%- endif -%}
  {% if item.minicourse %} (minicourse) {% endif %}\\
  <p style="color:gray">
  Instructors: {{ item.instructors }}. 
  {{ item.time }}{{ item.semesters }}.
  </p>
{% endfor %}
