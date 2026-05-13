---
layout: post
title: "博客搭建指南"
date: 2026-05-10
categories: tutorial
slug: blog-setup
nav_category: 博客搭建
nav_subcategory: 入门指南
nav_order: 1
---

## 环境要求

- Ruby 2.5 或更高版本
- RubyGems
- GCC 和 Make

## 安装 Jekyll

```bash
gem install jekyll bundler
jekyll new mysite
cd mysite
bundle exec jekyll serve
```

## 项目目录结构

```
my-jekyll-site/
├── _config.yml          # 站点配置文件
├── _posts/              # 博客文章目录
├── _layouts/            # 布局模板目录
├── _includes/           # 可复用的页面片段
├── _sass/               # Sass 样式文件
├── _site/               # 生成的静态网站（自动生成）
├── assets/              # 静态资源（CSS、JS、图片等）
├── Gemfile              # Ruby 依赖管理
├── Gemfile.lock         # 依赖版本锁定
├── index.markdown       # 首页
└── about.markdown       # 关于页面
```

## 核心文件详解

### _config.yml

Jekyll 最重要的配置文件，定义站点全局设置：

```yaml
title: 我的博客
description: 个人技术博客
url: "https://example.com"
baseurl: "/blog"
timezone: Asia/Shanghai
lang: zh-CN
markdown: kramdown
highlighter: rouge

plugins:
  - jekyll-feed
  - jekyll-seo-tag

defaults:
  - scope:
      path: "_posts"
      type: "posts"
    values:
      layout: "post"
```

### _posts/

存放博客文章，文件命名格式：`YYYY-MM-DD-title.markdown`

每篇文章开头需要 **Front Matter**：

```yaml
---
layout: post
title: "文章标题"
date: 2026-05-10
categories: tutorial
tags: [jekyll, web]
---
```

### _layouts/

定义页面整体结构，可被其他页面继承：

{% raw %}
```html
<!-- _layouts/default.html -->
<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang }}">
<head>
  <meta charset="utf-8">
  <title>{{ page.title }} | {{ site.title }}</title>
  {% include head.html %}
</head>
<body>
  {% include header.html %}
  <main>{{ content }}</main>
  {% include footer.html %}
</body>
</html>
```
{% endraw %}

### _includes/

存放可复用片段，通过 `{% raw %}{% include %}{% endraw %}` 引入。

### assets/

存放 CSS、JavaScript、图片、字体等静态资源。

## Liquid 模板语法

### 输出

{% raw %}
```liquid
{{ page.title }}
{{ site.title }}
{{ content }}
```
{% endraw %}

### 标签

{% raw %}
```liquid
{% if page.title %}
  <h1>{{ page.title }}</h1>
{% endif %}

{% for post in site.posts %}
  <a href="{{ post.url }}">{{ post.title }}</a>
{% endfor %}

{% assign my_var = "Hello" %}
```
{% endraw %}

### 过滤器

{% raw %}
```liquid
{{ page.date | date: "%Y-%m-%d" }}
{{ page.title | escape }}
{{ page.title | truncate: 20 }}
{{ site.posts | size }}
{{ "assets/style.css" | relative_url }}
```
{% endraw %}

## 常用命令

```bash
jekyll new my-site                    # 创建新站点
bundle exec jekyll serve              # 本地预览
bundle exec jekyll serve --port 4000 --host 0.0.0.0  # 指定端口和主机
jekyll build                          # 仅构建
jekyll build -d /path/to/output       # 构建到指定目录
jekyll clean                          # 清理生成文件
```

## 部署

### GitHub Pages

1. 将代码推送到 GitHub
2. 在仓库设置中启用 GitHub Pages
3. 选择分支和目录

自定义域名：在仓库根目录创建 `CNAME` 文件，写入域名。

### Netlify

1. 登录 Netlify，选择 Git 仓库
2. 构建设置：

```
Build command: bundle exec jekyll build
Publish directory: _site
```

### VPS 部署

1. 生成静态文件：

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

2. 配置 Nginx：

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/blog/_site;
    index index.html;
}
```

3. 自动化部署（Git hooks）：

```bash
#!/bin/bash
git --work-tree=/var/www/blog --git-dir=/var/repo/blog.git checkout -f
cd /var/www/blog && bundle exec jekyll build
```

### CDN 加速

使用 Cloudflare 等 CDN 服务：
1. 添加站点
2. 配置 DNS
3. 启用缓存规则
