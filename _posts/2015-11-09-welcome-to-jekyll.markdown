---
layout: post
title:  "Welcome to Jekyll!"
#date:   2015-11-09 15:39:40
preview: /static/images/common/300/300.jpg
#categories: [java, test]
---



大变量site.
site.posts: 所有博客的hash.

site.pages: 所有page, 元素的属性page和post应该没啥区别

site.related_posts: 显示相关联的10个博客,不知道这相关联是怎么取的, 目测是时间..

site.categories.CATEGORY: 显示这个分类下的博客列表

site.tags.TAG: 显示这个标签下的博客列表

post: site.posts单个值, 有url, title, excerpt等子属性, 注意excerpt就是预览文本,取前面一小段.

page: 这是针对单个文章的全局变量, 作用等同于post, 同样拥有url, title, excerpt等属性.

Paginator
貌似中文叫页码.其中提供了获取上一篇 和 下一篇的功能

paginator.per_page

paginator.posts

paginator.total_posts

paginator.total_pages

paginator.page

paginator.previous_page: 上一页

paginator.previous_page_path: 上一页路径

paginator.next_page: 下一页

paginator.next_page_path: 下一页路径

注意, 这个paginator只能在index文件中使用, 坑爹啊.

paginator原来是总的页数的意思, 可以在_config.yml中设置, 比如

paginate: 5
那就是5篇文章一页.

上一篇和下一篇的路径分别是用page.previous.url 和 page.next.url获取的.

jekyll的文档中貌似并没有说这俩个page的自属性.