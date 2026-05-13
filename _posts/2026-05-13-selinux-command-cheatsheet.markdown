---
layout: post
title:  "SELinux 工具速查手册"
date:   2026-05-13 21:00:00 +0800
categories: linux security
tags: [linux, selinux, security, mac]
nav_category: Linux
nav_subcategory: 系统与安全
nav_order: 1
---

SELinux（Security-Enhanced Linux）是 Linux 上最主流的强制访问控制（MAC）机制，但相关命令繁多，经常需要翻文档。这里整理了一份速查表，覆盖日常运维和策略开发中最常用的命令。

## 状态管理

| 命令 | 作用 | 用法示例 | 备注 |
|------|------|----------|------|
| `getenforce` | 查看当前模式 | `getenforce` | 输出 `Enforcing`、`Permissive` 或 `Disabled` |
| `setenforce` | 临时切换模式 | `setenforce 0`（宽松）/ `setenforce 1`（强制） | 重启后失效 |
| `sestatus` | 显示详细状态 | `sestatus` | 包括策略、模式、文件系统状态等 |

## 安全上下文

| 命令 | 作用 | 用法示例 | 备注 |
|------|------|----------|------|
| `ls -Z` | 查看文件安全上下文 | `ls -Z /var/www/html` | `-Z` 显示 SELinux 上下文 |
| `ps -efZ` | 查看进程安全上下文 | `ps -efZ \| grep httpd` | |
| `chcon` | 临时修改上下文 | `chcon -t httpd_sys_content_t /file` | 下次重新打标签后失效 |
| `semanage fcontext` | 永久定义上下文规则 | `semanage fcontext -a -t httpd_t "/var/www(/.*)?"` | 需配合 `restorecon` |
| `restorecon` | 恢复默认上下文 | `restorecon -Rv /var/www/` | `-R` 递归，`-v` 显示详情 |

## 布尔值管理

| 命令 | 作用 | 用法示例 | 备注 |
|------|------|----------|------|
| `getsebool` | 查看布尔值状态 | `getsebool httpd_can_network_connect` | 检查策略开关 |
| `setsebool` | 修改布尔值 | `setsebool -P httpd_can_network_connect on` | `-P` 永久生效 |

## 故障排除

| 命令 | 作用 | 用法示例 | 备注 |
|------|------|----------|------|
| `audit2allow` | 根据审计日志生成策略 | `cat /var/log/audit/audit.log \| audit2allow` | 将拒绝行为转为允许规则 |
| `ausearch` | 搜索审计日志 | `ausearch -c 'httpd' -m 'AVC'` | 常与 `audit2allow` 配合 |

## 端口管理

| 命令 | 作用 | 用法示例 |
|------|------|----------|
| `semanage port` | 管理端口安全上下文 | `semanage port -a -t http_port_t -p tcp 8080` |

## 用户映射

| 命令 | 作用 | 用法示例 |
|------|------|----------|
| `semanage login` | 管理 Linux 用户与 SELinux 用户映射 | `semanage login -a -s staff_u myuser` |

## 策略开发

| 命令 | 作用 | 用法示例 | 备注 |
|------|------|----------|------|
| `checkmodule` | 编译策略源码 | `checkmodule -M -m -o mymodule.mod mymodule.te` | `.te` → `.mod` |
| `semodule_package` | 打包策略模块 | `semodule_package -o mymodule.pp -m mymodule.mod` | `.mod` → `.pp` |
| `semodule` | 加载/卸载策略模块 | `semodule -i mymodule.pp`（安装）/ `semodule -r mymodule`（移除） | |
| `seinfo` | 查询策略信息 | `seinfo -t`（查类型）/ `seinfo -t httpd_t`（查特定类型） | 调试利器 |
| `sesearch` | 查询策略规则 | `sesearch -A -s httpd_t -t httpd_sys_content_t -c file` | 查看特定访问权限 |

---

> 💡 **小贴士**：开发自定义策略时，典型流程是 `checkmodule` → `semodule_package` → `semodule -i`。排障时先看 `audit2allow` 的输出，理解拒绝原因后再决定是写策略还是调布尔值。
