---
layout: post
title: "插件开发"
date: 2026-05-10
categories: tutorial
slug: plugin-development
nav_category: 配置与定制
nav_subcategory: 高级配置
nav_order: 2
---

## Jekyll 插件系统

Jekyll 插件可以扩展站点功能。

## 插件类型

### Generators

生成器用于创建新文件：

```ruby
Jekyll::Hooks.register :site, :after_init do |site|
  # 在站点初始化后执行
end
```

### Converters

转换器处理文件格式转换：

```ruby
class MyConverter < Jekyll::Converter
  def matches(ext)
    ext == '.my'
  end

  def convert(content)
    # 转换内容
  end
end
```

### Tags

自定义 Liquid 标签：

```ruby
class RandomTag < Liquid::Tag
  def render(context)
    rand(100).to_s
  end
end

Liquid::Template.register_tag('random', RandomTag)
```

## 实用插件示例

### 阅读时间

```ruby
module Jekyll
  module ReadingTime
    def reading_time(content)
      words = content.split.size
      minutes = (words / 200.0).ceil
      "#{minutes} min read"
    end
  end
end

Liquid::Template.register_filter(Jekyll::ReadingTime)
```

使用方法：

```liquid
{{ content | reading_time }}
```

## 发布插件

1. 创建 gem 包
2. 发布到 RubyGems
3. 添加文档和示例
