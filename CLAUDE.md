# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll static site (documentation-style blog) built with Ruby. It uses the minima theme with heavy customization, featuring a documentation-style layout with sidebar navigation.

## Development Commands

```bash
# Install dependencies
bundle install

# Start local development server (http://127.0.0.1:4000)
bundle exec jekyll serve

# Build site to _site/ directory
bundle exec jekyll build

# Clean generated files
bundle exec jekyll clean
```

## Architecture

### Layout Hierarchy

```
docs.html (base layout with sidebar)
  └── post.html (wraps content in docs layout)
      └── Article content
```

- `docs.html` - Main documentation layout with left sidebar navigation
- `home.html` - Homepage with tag-filterable post cards
- `post.html` - Simple wrapper that sets `layout: docs`

### Navigation System

Posts define navigation structure via front matter:

```yaml
nav_category: 博客搭建        # Top-level category (required for sidebar)
nav_subcategory: 入门指南     # Optional second-level grouping
nav_order: 1                  # Sort order within category
```

The sidebar is auto-generated from posts with `nav_category` defined. Posts without `nav_category` appear in an "未分组" (Ungrouped) section.

### Includes

- `head.html` - HTML head with Mermaid and KaTeX CDN scripts
- `header.html` - Site header with mobile menu toggle
- `footer.html` - Site footer

### Key Features

- **Mermaid diagrams**: Loaded via CDN in `head.html`
- **KaTeX math**: Loaded via CDN for LaTeX rendering
- **Tag filtering**: Homepage JavaScript filters posts by tags
- **Mobile-responsive sidebar**: Toggle menu for mobile view

### Post File Naming

Files in `_posts/` must follow: `YYYY-MM-DD-slug.markdown`

### Styles

- `assets/main.scss` - Main stylesheet, imports minima theme then overrides
- Uses LXGW WenKai Mono font (local file in `assets/fonts/`)
