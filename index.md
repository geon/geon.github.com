---
title: Hello World!
tagline: I wish I knew what I'm doing.
---

I have set up a blog to publish the stuff on my mind that are too long for a facebook status. We'll see how that goes.
    
## Posts so far...

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
