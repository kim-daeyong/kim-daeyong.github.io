---
layout: page
title: log
---

{% assign postsByMonth = site.posts | group_by_exp:"post", "post.date | date: '%b %Y'" %}

<div class="timeline_introduce">현재까지 {{ site.posts.size }}개의 포스트를 작성했어요.</div>

{% for month in postsByMonth %}

<h1 style="color:green; border-bottom:2px solid green" id="{{ month.name | slugify }}" class="archive__subtitle">{{ month.name }}</h1>

{% for post in month.items %}

<div class="posts-list">
    <article class="post-preview">
        <a href="{{ post.url | prepend: site.baseurl }}">
            <h3 class="post-title">{{ post.title }}</h3>
            {% if post.subtitle %}
            <h3 class="post-subtitle">
                {{ post.subtitle }}
            </h3>
            {% endif %}
        </a>
        <p class="post-meta">
            Posted on {{ post.date | date: "%B %-d, %Y" }}
        </p>
        <div class="post-entry-container">
            {% if post.image %}
            <div class="post-image">
                <a href="{{ post.url | prepend: site.baseurl }}">
                    <img src="{{ post.image }}">
                </a>
            </div>
            {% endif %}
            <div class="post-entry">
                {{ post.excerpt | strip_html | xml_escape | truncatewords: site.excerpt_length }}
                {% assign excerpt_word_count = post.excerpt | number_of_words %}
                {% if post.content != post.excerpt or excerpt_word_count > site.excerpt_length %}
                <a href="{{ post.url | prepend: site.baseurl }}" class="post-read-more">[Read&nbsp;More]</a>
                {% endif %}
            </div>
        </div>
    </article>
</div>
  {% endfor %}
{% endfor %}

