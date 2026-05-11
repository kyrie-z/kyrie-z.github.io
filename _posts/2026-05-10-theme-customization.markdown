---
layout: post
title: "主题定制"
date: 2026-05-10
categories: tutorial
slug: theme-customization
---

## 主题系统

Jekyll 支持灵活的主题定制。

### 默认主题

Minima 是 Jekyll 的默认主题，提供简洁的设计。

### 覆盖主题文件

你可以通过创建同名文件来覆盖主题中的文件：

- `assets/main.scss` - 自定义样式
- `_layouts/default.html` - 自定义布局
- `_includes/header.html` - 自定义头部

## 样式定制

### 变量覆盖

在 `assets/main.scss` 中覆盖主题变量：

```scss
---
---

$brand-color: #2196F3;
$text-color: #333;

@import "minima";
```

### 自定义 CSS

添加额外的样式规则：

```scss
.post-content h2 {
  border-bottom: 2px solid $brand-color;
}
```

## 布局定制

### 修改默认布局

复制主题布局到 `_layouts/` 目录进行修改：

```bash
bundle show minima
cp $(bundle show minima)/_layouts/*.html _layouts/
```

### 创建新布局

```html
---
layout: default
---
<article class="post">
  <header class="post-header">
    <h1>{{ page.title }}</h1>
  </header>
  <div class="post-content">
    {{ content }}
  </div>
</article>
```

## JavaScript 增强

### 添加交互功能

```javascript
document.addEventListener('DOMContentLoaded', function() {
  // 你的代码
});
```
