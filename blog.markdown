---
layout: page
title: Blog
permalink: /blog/
---

I blog under two conditions:

- When I am extremely motivated about something
- When I feel guilty that i haven't blogged in a while. 

If you don't believe me, checkout the wild inconsistency of the [timeline](#timeline). But also check out the content. It is (mostly) nice.

#### Featured

Add featured posts here with images

#### Timeline

{% assign postsByYear = site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}

{% for year in postsByYear %}
  <h5>{{ year.name }}</h5>
  <ul>
  {% for post in year.items %}
    <li>
      <p>{{ post.date | date: "%-d %B %Y" }} | <a href="{{ post.url }}">{{ post.title }}</a></p>
    </li>
  {% endfor %}
  </ul>
{% endfor %}