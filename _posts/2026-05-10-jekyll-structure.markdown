---
layout: post
title: "Jekyll 目录结构与文件说明"
date: 2026-05-10
categories: tutorial
slug: jekyll-structure
nav_category: 入门指南
nav_order: 2
---

## Jekyll 项目目录结构

一个典型的 Jekyll 项目目录结构如下：

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

### _config.yml - 站点配置文件

这是 Jekyll 最重要的配置文件，用于定义站点的全局设置：

```yaml
# 站点基本信息
title: 我的博客              # 站点标题
description: 个人技术博客    # 站点描述
author: 张三                 # 作者名称
email: example@mail.com      # 联系邮箱

# 站点 URL 设置
url: "https://example.com"   # 站点完整 URL
baseurl: "/blog"             # 子目录路径（如果没有则留空）

# 时区和语言
timezone: Asia/Shanghai
lang: zh-CN

# 构建设置
markdown: kramdown           # Markdown 解析器
highlighter: rouge           # 代码高亮器

# 插件
plugins:
  - jekyll-feed              # RSS 订阅
  - jekyll-seo-tag           # SEO 优化

# 默认值设置（为特定类型文件设置默认 front matter）
defaults:
  - scope:
      path: "_posts"
      type: "posts"
    values:
      layout: "post"
      author: "张三"

# 自定义变量（可在模板中通过 site.变量名 访问）
nav_menu:
  - name: "入门指南"
    items:
      - quick-start
      - jekyll-structure
```

### _posts/ - 博客文章目录

存放所有博客文章，文件命名必须遵循格式：`YYYY-MM-DD-title.markdown`

```
_posts/
├── 2026-05-10-welcome-to-jekyll.markdown
├── 2026-05-10-quick-start.markdown
└── 2026-05-10-advanced-tips.markdown
```

每篇文章开头需要有 **Front Matter**（YAML 格式的元数据块）：

```markdown
---
layout: post           # 使用的布局模板
title: "文章标题"       # 文章标题
date: 2026-05-10       # 发布日期
categories: tutorial   # 分类（可多个）
tags: [jekyll, web]    # 标签（可多个）
slug: my-article       # URL 别名（用于生成链接）
---

## 正文开始

这里是文章内容...
```

### _layouts/ - 布局模板目录

定义页面的整体结构，可以被其他页面继承：

{% raw %}
```html
<!-- _layouts/default.html - 基础布局 -->
<!DOCTYPE html>
<html lang="{{ page.lang | default: site.lang }}">
<head>
  <meta charset="utf-8">
  <title>{{ page.title }} | {{ site.title }}</title>
  {% include head.html %}
</head>
<body>
  {% include header.html %}
  <main>
    {{ content }}  <!-- 子模板的内容会插入这里 -->
  </main>
  {% include footer.html %}
</body>
</html>
```

```html
<!-- _layouts/post.html - 文章布局（继承 default） -->
---
layout: default
---
<article class="post">
  <header>
    <h1>{{ page.title }}</h1>
    <time>{{ page.date | date: "%Y-%m-%d" }}</time>
  </header>
  <div class="content">
    {{ content }}
  </div>
</article>
```
{% endraw %}

### _includes/ - 可复用片段

存放可在多个页面中复用的小片段：

```
_includes/
├── head.html       # HTML <head> 部分
├── header.html     # 页面顶部导航
├── footer.html     # 页面底部
├── sidebar.html    # 侧边栏
└── share.html      # 分享按钮组件
```

{% raw %}
使用 `{% include %}` 标签引入：

```html
{% include header.html %}
{% include sidebar.html category="tutorial" %}
```
{% endraw %}

### assets/ - 静态资源

存放 CSS、JavaScript、图片等静态文件：

```
assets/
├── css/
│   └── main.scss        # 主样式文件
├── js/
│   └── main.js          # 主脚本文件
├── images/
│   └── logo.png         # 图片资源
└── fonts/
    └── custom-font.ttf  # 自定义字体
```

### Gemfile - Ruby 依赖管理

定义项目所需的 Ruby gems：

```ruby
source "https://rubygems.org"

# Jekyll 版本
gem "jekyll", "~> 4.3"

# 主题
gem "minima", "~> 2.5"

# 插件
group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
end

# 开发环境
gem "webrick", "~> 1.7"
```

安装依赖：

```bash
bundle install
```

## Jekyll 模板语法

Jekyll 使用 Liquid 模板语言，主要有三种类型：

### 1. 输出 `&#123;&#123; &#125;&#125;`

输出变量的值：

{% raw %}
```liquid
{{ page.title }}        <!-- 页面标题 -->
{{ site.title }}        <!-- 站点标题 -->
{{ content }}           <!-- 页面内容 -->
```
{% endraw %}

### 2. 标签 `&#123;&#37; &#37;&#125;`

执行逻辑操作：

{% raw %}
```liquid
<!-- 条件判断 -->
{% if page.title %}
  <h1>{{ page.title }}</h1>
{% endif %}

<!-- 循环 -->
{% for post in site.posts %}
  <a href="{{ post.url }}">{{ post.title }}</a>
{% endfor %}

<!-- 赋值 -->
{% assign my_var = "Hello" %}
```
{% endraw %}

### 3. 过滤器 `|`

对输出进行处理：

{% raw %}
```liquid
<!-- 日期格式化 -->
{{ page.date | date: "%Y-%m-%d" }}

<!-- 文本处理 -->
{{ page.title | escape }}        <!-- HTML 转义 -->
{{ page.title | truncate: 20 }}  <!-- 截断文本 -->

<!-- 数组操作 -->
{{ site.posts | size }}          <!-- 获取数量 -->
{{ site.posts | first }}         <!-- 获取第一个 -->

<!-- 链接处理 -->
{{ "assets/style.css" | relative_url }}  <!-- 相对路径 -->
{{ page.url | absolute_url }}             <!-- 绝对路径 -->
```
{% endraw %}

## 常用命令

```bash
# 创建新站点
jekyll new my-site

# 本地预览（自动刷新）
bundle exec jekyll serve

# 指定端口和主机
bundle exec jekyll serve --port 4000 --host 0.0.0.0

# 仅构建不预览
jekyll build

# 构建到指定目录
jekyll build -d /path/to/output

# 清理生成的文件
jekyll clean
```

## 总结

| 文件/目录 | 作用 |
|-----------|------|
| `_config.yml` | 站点全局配置 |
| `_posts/` | 博客文章存放处 |
| `_layouts/` | 页面布局模板 |
| `_includes/` | 可复用的页面片段 |
| `assets/` | 静态资源文件 |
| `Gemfile` | Ruby 依赖管理 |
| `_site/` | 生成的静态网站（勿手动修改） |

理解这些核心文件的作用，是使用 Jekyll 的基础。接下来可以学习如何自定义主题和开发插件。
