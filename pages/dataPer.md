---
layout: page
title: 数据持久层 系列文章
titlebar: 数据持久层
subtitle: <span class="mega-octicon octicon-clippy"></span>&nbsp;&nbsp; 数据持久层 系列教程
menu: dataPer
css: ['blog-page.css']
permalink: /dataPer
keywords: 数据持久层配置应用,Mybatis,Ibatis,Driud,JDBC等
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='dataPer'  or post.keywords contains 'dataPer' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
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
