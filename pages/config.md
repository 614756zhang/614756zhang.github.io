---
layout: page
title: 配置详解 系列文章
titlebar: 配置详解
subtitle: <span class="mega-octicon octicon-flame"></span>&nbsp;&nbsp; 配置详解 系列教程
menu: config
css: ['blog-page.css']
permalink: /config
keywords: 配置详解
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='config'  or post.keywords contains 'config' or post.keywords contains 'config' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">{{ post.title }}</a>
                        <span class='circle'></span>
                    </div>
                </li>
                {% endif %}
            {% endfor %}
        </ul> 

        <!-- Pagination -->
        {% include pagination.html %}

        <!-- Comments -->
       <div class="comment">
         {% include comments.html %}
       </div>
    </div>

</div>
<script>
    $(document).ready(function(){

        // Enable bootstrap tooltip
        $("body").tooltip({ selector: '[data-toggle=tooltip]' });

    });
</script>
