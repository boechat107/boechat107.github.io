---
layout: page
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts offset: 0 limit: 20 %}
<div class="row"> 
{% if post.image %}
    <div class="span2">
    <a href="{{ post.url }}" >
        <img border="0" width="250" height="150" src="/img/posts/{{ post.image }}" alt="">
    </a>
    </div>
{% endif %}
    <!--<h4><strong><a href="{{ post.url }}">{{ post.title }}</a></strong></h4>-->
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p> {{ post.description }} </p>
    <p>
    <i class="icon-calendar"></i> {{ post.date | date: "%B %e, %Y" }}
    | <i class="icon-tags"></i> Tags :{% for tag in post.tags %} <a href="/tags/{{ tag }}" rel="tooltip" title="View posts tagged with &quot;{{ tag }}&quot;"><span class="label label-info">{{ tag }}</span></a>  {% if forloop.last != true %} {% endif %} {% endfor %}
    </p>
    <p><a href="{{ post.url }}">Read more</a></p> 
</div>
<br>
{% endfor %}
