---
layout: post
title: "快速入门指南"
date: 2026-05-10
categories: tutorial
slug: quick-start
nav_category: 入门指南
nav_order: 1
---

## 简介

这是一个快速入门指南，帮助你快速上手 Jekyll 静态网站。

## 安装步骤

### 环境要求

- Ruby 2.5 或更高版本
- RubyGems
- GCC 和 Make

### 安装 Jekyll

```bash
gem install jekyll bundler
jekyll new mysite
cd mysite
bundle exec jekyll serve
```

## 目录结构

Jekyll 项目的基本目录结构如下：

- `_posts/` - 博客文章
- `_layouts/` - 布局模板
- `_includes/` - 可复用组件
- `_config.yml` - 配置文件

## 下一步

阅读 [进阶配置](/jekyll/update/2026/05/10/advanced-configuration.html) 了解更多自定义选项。
