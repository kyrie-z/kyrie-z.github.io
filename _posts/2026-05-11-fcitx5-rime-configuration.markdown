---
layout: post
title:  "fcitx5-rime 输入法配置"
date:   2026-05-11 21:00:00 +0800
categories: linux input-method
tags: [linux, fcitx, rime]
slug: fcitx5-rime雾凇拼音
nav_category: Linux
nav_subcategory: 桌面与应用
nav_order: 1
---

## 前言：Rime 输入法简介

Rime 全称为「Rime 中州韵输入法引擎」，通常简称为 Rime 或中州韵。

- 官网：[RIME \| 中州韻輸入法引擎](https://rime.im/)
- 特点：
  - 提供核心框架，支持 Windows、macOS、Linux、Android、iOS 等多平台
  - 无云端服务器，全部词库和配置均在本地
  - 使用 YAML 格式配置文件，可自定义候选框皮肤、词库、输入逻辑等

---

## 一、fcitx5-rime 安装与启用

### 1.1 安装

```bash
sudo apt install fcitx5 fcitx5-configtool fcitx5-rime
```

如果是 GNOME 环境，还需安装：

```bash
sudo apt install fcitx5-frontend-gtk3 fcitx5-frontend-gtk4
```

### 1.2 配置环境变量

编辑 `/etc/environment` 文件，添加以下内容：

```bash
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS="@im=fcitx"
```

### 1.3 设置自启动

在 GNOME 中：
1. 打开「优化」（tweaks）
2. 进入「开机启动程序」
3. 添加 `fcitx5`

### 1.4 启用 Rime 输入法

1. 在系统托盘右键点击「小企鹅」图标，或在终端执行：
   ```bash
   fcitx5-configtool
   ```
2. 进入配置界面后，将右面板「可用输入法」中的 **中州韵** 或 **rime** 添加到左侧「当前输入法」
3. 点击「应用」即可启用

> **提示**：此时 Rime 还是基础版本，建议继续配置雾凇拼音以获得更好的体验。

---

## 二、雾凇拼音配置

### 2.1 安装

#### 方案一：手动安装

1. 下载 [雾凇拼音](https://github.com/iDvel/rime-ice/releases)
2. 解压，复制并覆盖所有文件到输入法「用户目录」：
   ```bash
   ~/.local/share/fcitx5/rime
   ```
3. 使用「中州韵」输入法，调出输入框后按快捷键 **F4** 或 **Ctrl+\`** 选择「雾凇拼音」

#### 方案二：使用 plum 配方管理工具（推荐）

```bash
# 下载 plum
git clone https://github.com/rime/plum.git

# 进入目录并安装
cd plum
```

选择以下安装方式：

```bash
# 默认安装
bash rime-install iDvel/rime-ice

# 或为 fcitx5 安装
rime_frontend=fcitx5-rime bash rime-install iDvel/rime-ice
```

> 更多详细指南参考：[安装文档](https://github.com/iDvel/rime-ice/blob/main/others/docs/Installation.md)

执行后配置文件默认位于 `~/.local/share/fcitx5/rime` 目录下。

---

### 2.2 配置（可选）

> 以下配置根据个人使用习惯按需设置。

配置目录：`~/.local/share/fcitx5/rime`

#### 开启逗号、句号翻页

修改 `default.yaml` 或 `default.custom.yaml`：

```bash
sed -i 's/# \(- { when: \(paging\|has_menu\), accept: \(comma\|period\), send: Page_\(Up\|Down\) }\)/\1/' default.yaml
```

#### 更改候选词数量

```bash
sed -i 's/page_size: 5/page_size: 9/' default.yaml
```

---

## 三、万象拼音大模型配置

GitHub 地址：[rime_wanxiang](https://github.com/amzxyz/rime_wanxiang)

### 3.1 安装

下载 GitHub Release 中最新的语法模型文件：`wanxiang-lts-zh-hans.gram`

### 3.2 配置

1. 将下载的 `wanxiang-lts-zh-hans.gram` 放到输入法用户目录根目录：
   ```bash
   ~/.local/share/fcitx5/rime
   ```

2. 新建或编辑 `rime_ice.custom.yaml` 文件，添加以下内容：

   ```yaml
   patch:
     grammar:
       language: wanxiang-lts-zh-hans
       collocation_max_length: 5
       collocation_min_length: 2
     translator/contextual_suggestions: true
     translator/max_homophones: 7
     translator/max_homographs: 7
   ```

3. 重启 fcitx5 使配置生效

### 3.3 验证生效

检测是否生效可参考项目的 README 说明和介绍。

---

## 四、注意事项

> **重要**：以上所有配置修改后，都需要重新启动 fcitx5 才能生效。
