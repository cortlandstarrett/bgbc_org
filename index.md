---
layout: default
title: Home
permalink: /
---

<section class="hero">
  <h1 class="hero__brand">{{ site.data.site.name }}</h1>
  <p class="hero__lede">{{ site.data.site.tagline }}</p>
  <div class="hero__actions">
    <a class="button" href="{{ '/sermons/' | relative_url }}">Listen to sermons</a>
    <a class="button button--secondary" href="{{ '/ministries/' | relative_url }}">Ministries</a>
  </div>
</section>

<section class="home-section" aria-labelledby="service-times-heading">
  <h2 id="service-times-heading">Service times</h2>
  <ul class="service-times">
    {% for item in site.data.site.service_times %}
    <li>
      <strong>{{ item.label }}</strong>
      <span>{{ item.detail }}</span>
    </li>
    {% endfor %}
  </ul>
</section>

<section class="home-section" aria-labelledby="welcome-heading">
  <h2 id="welcome-heading">Welcome</h2>
  <div class="prose">
    <p>
      We gather to worship God, grow in the Word, and care for one another.
      Browse recent sermons, meet our staff, or learn what we believe.
    </p>
  </div>
</section>
