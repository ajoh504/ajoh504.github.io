{% if post.content contains site.excerpt_separator %}
  {{ post.excerpt }}
  <a href="{{ post.url }}">Read more</a>
{% else %}
  {{ post.content }}
{% endif %}