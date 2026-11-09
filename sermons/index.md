---
title: Sermons
permalink: /sermons/
layout: page
---

Browse recent messages below. Audio will be attached as files are migrated; until then, sermon pages show metadata and a clear “not available yet” note.

{% assign sermons = site.sermons | sort: 'date' | reverse %}
{% if sermons.size == 0 %}
<p>No sermons have been published yet.</p>
{% else %}
<ul class="sermon-list">
  {% for sermon in sermons %}
  <li>
    <a href="{{ sermon.url | relative_url }}">{{ sermon.title }}</a>
    <span class="meta">
      {{ sermon.date | date: '%B %-d, %Y' }}
      {% if sermon.scripture %} · {{ sermon.scripture }}{% endif %}
      {% if sermon.speaker %} · {{ sermon.speaker }}{% endif %}
      {% unless sermon.audio %} · audio pending{% endunless %}
    </span>
  </li>
  {% endfor %}
</ul>
{% endif %}
