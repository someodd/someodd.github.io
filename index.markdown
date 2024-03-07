---
layout: page
excerpt: "someodd"
show_excerpts: true
paginate: true
entries_layout: grid
---

{% assign combined_collections = site.posts | concat: site.showcase %}
{% assign sorted_posts = combined_collections | sort: 'date' | reverse %}

<div class="entries-{{ page.entries_layout | default: 'list' }}">
  {% for entry in sorted_posts %}
    {% include entry.html %}
  {% endfor %}
</div>
