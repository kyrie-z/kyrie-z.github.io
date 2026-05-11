---
layout: post
title: "部署指南"
date: 2026-05-10
categories: tutorial
slug: deployment-guide
---

## 部署选项

Jekyll 站点可以部署到多种平台。

## GitHub Pages

### 自动部署

1. 将代码推送到 GitHub
2. 在仓库设置中启用 GitHub Pages
3. 选择分支和目录

### 自定义域名

在仓库根目录创建 `CNAME` 文件：

```
www.example.com
```

## Netlify

### 连接仓库

1. 登录 Netlify
2. 选择 Git 提供商
3. 选择仓库

### 构建设置

```
Build command: bundle exec jekyll build
Publish directory: _site
```

## VPS 部署

### 使用 Nginx

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

### 自动化部署

使用 Git hooks 实现自动部署：

```bash
#!/bin/bash
git --work-tree=/var/www/blog --git-dir=/var/repo/blog.git checkout -f
cd /var/www/blog && bundle exec jekyll build
```

## CDN 加速

### Cloudflare

1. 添加站点
2. 配置 DNS
3. 启用缓存规则

### 性能检查

使用 PageSpeed Insights 检查性能优化效果。
