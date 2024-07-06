---
layout: page
title: List of posts
---

<ul class="list-none p-0">
    {%- for post in site.posts -%}
        <li>
            <span>{{ post.date | date: "%Y-%m-%d" }}</span> - <a href="{{ post.url }}">{{ post.title }}</a>
        </li>
    {%- endfor -%}
</ul>
