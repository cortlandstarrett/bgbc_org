---
title: Staff
permalink: /staff/
---

## Senior Pastor & Elders

{% for person in site.data.people.elders %}
{% include person.html person=person %}
{% endfor %}

## Deacons

{% for person in site.data.people.deacons %}
{% include person.html person=person %}
{% endfor %}

## Trustees

{% for person in site.data.people.trustees %}
{% include person.html person=person %}
{% endfor %}

## Support Staff

{% for person in site.data.people.support_staff %}
{% include person.html person=person %}
{% endfor %}
