---
layout: default
title: Home
permalink: /
---

<section class="hero hero--full-bleed">
  <h1 class="visually-hidden">{{ site.data.site.name }}</h1>
  <img class="hero__photo" src="{{ '/assets/images/hero-front.jpg' | relative_url }}" alt="Battle Ground Bible Church building" width="1200" height="675" loading="eager">
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
