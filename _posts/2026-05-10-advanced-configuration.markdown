---
layout: post
title: "进阶配置"
date: 2026-05-10
categories: tutorial
slug: advanced-configuration
---

## 配置文件详解

`_config.yml` 是 Jekyll 的核心配置文件。

### 基本设置

```yaml
title: 我的博客
description: 一个技术博客
url: "https://example.com"
```

### 构建设置

```yaml
markdown: kramdown
highlighter: rouge
permalink: /:categories/:year/:month/:day/:title/
```

## 自定义布局

### 创建布局文件

在 `_layouts/` 目录下创建 HTML 文件：

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{ page.title }}</title>
</head>
<body>
  {{ content }}
</body>
</html>
```

### 使用布局

在文章头部指定布局：

```yaml
---
layout: post
title: 我的文章
---
```

## 插件使用

### 常用插件

- `jekyll-seo-tag` - SEO 优化
- `jekyll-sitemap` - 站点地图
- `jekyll-feed` - RSS 订阅

### 安装插件

1. 在 Gemfile 中添加：

```ruby
gem 'jekyll-seo-tag'
```

2. 在 `_config.yml` 中启用：

```yaml
plugins:
  - jekyll-seo-tag
```

## 性能优化

### 图片优化

- 使用 WebP 格式
- 添加 lazy loading
- 使用 CDN 加速

### 代码分割

- 异步加载 JavaScript
- 内联关键 CSS
