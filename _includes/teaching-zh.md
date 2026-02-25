{% for item in site.data.teaching-zh %}
- **{{ item.role }}：**
  {% if item.site %}[{{ item.course }}]({{item.site}})
  {%- else -%}{{ item.course }}
  {%- endif -%}
  {% if item.minicourse %}（微课程）{% endif %}\\
  <p style="color:gray">
  任课教师：{{ item.instructors }}。
  {{ item.time }}{{ item.semesters }}
  </p>
{% endfor %}
