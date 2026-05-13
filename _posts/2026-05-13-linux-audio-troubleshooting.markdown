---
layout: post
title:  "Linux 音频排查"
date:   2026-05-13 21:30:00 +0800
categories: linux desktop
tags: [linux, security, pipewire, audio]
nav_category: Linux
nav_subcategory: 桌面与应用
nav_order: 2
---

Linux 桌面的音频栈从 ALSA → PulseAudio → PipeWire 演进了很多，但偶尔还是会遇到无声、设备识别异常等问题。本文记录了一套系统的排查流程，适用于 PipeWire + WirePlumber 环境。

## 诊断步骤

### 1. 硬件检测

```bash
# 检查 PCI 音频设备
lspci -v | grep -i audio

# 检查 ALSA 播放设备
aplay -l
```

### 2. 驱动状态检查

```bash
# 检查内核驱动
lsmod | grep snd

# 检查 ALSA 设备列表
aplay -L
```

### 3. ALSA 状态检查

```bash
# 检查混音器设置
amixer
amixer scontrols

# 检查主音量
amixer get Master
```

**关键检查点：**
- Master 音量是否开启（显示 `[on]`）
- 音量百分比是否大于 0
- 是否有多个输出设备

### 4. PipeWire / WirePlumber 状态检查

```bash
# 检查 PipeWire 服务状态
systemctl --user status pipewire pipewire-pulse

# 检查 WirePlumber 服务状态
systemctl --user status wireplumber

# 查看音频节点状态
wpctl status

# 查看详细设备信息
pw-dump | jq '.[] | select(.info.props."media.class" == "Audio/Sink") | .info.props'
```

### 5. 日志分析

```bash
# WirePlumber 日志
journalctl --user -u wireplumber --since "1 hour ago" | tail -50

# 内核音频消息
dmesg | grep -i "audio\|sound\|snd\|sof\|alc" | tail -30
```

## 常见问题模式

### 模式1：虚拟输出而非硬件输出

**症状：** `wpctl status` 显示默认输出为"虚拟输出"

```bash
Audio
 ├─ Sinks:
 │  *   35. 虚拟输出    [vol: 0.71]
```

**原因：** WirePlumber 未能成功创建硬件音频节点。

### 模式2：WirePlumber 节点创建失败

**日志示例：**

```
s-monitors: Failed to create alsa_output.pci-0000_00_1f.3-platform-skl_hda_dsp_generic.HiFi__Speaker__sink: Object activation aborted: PipeWire proxy destroyed
```

**可能原因：**
- WirePlumber 配置问题
- PipeWire 与 ALSA 通信故障
- 权限不足
- 内核驱动问题

### 模式3：设备静音或音量过低

```bash
wpctl get-volume @DEFAULT_AUDIO_SINK@
wpctl inspect <device_id> | grep -i mute
```

### 模式4：默认输出设备错误

```bash
wpctl status
# 确认默认输出是否为硬件设备
```

## 解决方案

### 方案1：重启音频服务

```bash
# 重启 WirePlumber（最常用）
systemctl --user restart wireplumber

# 重启整个 PipeWire 栈
systemctl --user restart pipewire pipewire-pulse wireplumber

# 等待后检查
sleep 3 && wpctl status
```

### 方案2：设置正确的默认输出

```bash
# 列出所有音频输出
wpctl status | grep -A5 "Sinks:"

# 设置硬件设备为默认输出
wpctl set-default <device_id>

# 取消静音
wpctl set-mute <device_id> 0

# 调整音量（0.0-1.0）
wpctl set-volume <device_id> 0.8
```

### 方案3：直接测试 ALSA 输出

```bash
# 绕过 PipeWire 直接测试硬件
speaker-test -D hw:0,0 -c 2 -t sine -f 440
```

### 方案4：检查权限和组

```bash
groups | grep audio
ls -la /dev/snd/
sudo usermod -aG audio $USER
```

## 深度分析：开机无声问题

### 问题现象

每次开机后无声音，需手动重启 WirePlumber 才能恢复。

### 关键日志证据

1. **WirePlumber 启动失败**：
   ```
   s-monitors: Failed to create alsa_output...: Object activation aborted: PipeWire proxy destroyed
   ```

2. **PipeWire 设备忙错误**：
   ```
   spa.alsa: '_ucm0001.hw:sofhdadsp,5': playback open failed: 设备或资源忙
   ```

### 根本原因

**ALSA 设备初始化与 PipeWire 服务启动的时序竞争：**

1. 开机序列：内核加载驱动 → `alsa-restore.service` 恢复状态 → 用户会话启动 PipeWire/WirePlumber
2. 竞争窗口：WirePlumber 启动时，ALSA 设备可能仍在固件加载或被 `alsa-restore` 短暂占用
3. 失败后果：PipeWire 首次打开设备失败 → WirePlumber 销毁设备对象 → 系统回退到虚拟输出

**重启 WirePlumber 有效的原理**：重启时 ALSA 设备已完全就绪，无竞争条件。

### 验证方法

```bash
# 查看完整启动时间线
journalctl -b --user-unit=pipewire --user-unit=wireplumber

# 检查系统服务时序
systemd-analyze plot > boot-timeline.svg

# 查看内核音频初始化
dmesg | grep -E "(sof|snd|audio|hda)" | head -30
```

## 命令速查表

| 命令 | 用途 | 示例 |
|------|------|------|
| `wpctl status` | 查看音频节点 | `wpctl status` |
| `wpctl set-default` | 设置默认输出 | `wpctl set-default 51` |
| `wpctl set-volume` | 调整音量 | `wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.8` |
| `wpctl set-mute` | 静音/取消 | `wpctl set-mute 51 0` |
| `amixer` | ALSA 混音器 | `amixer sset Master 80%` |
| `aplay -l` | 列出播放设备 | `aplay -l` |
| `pw-cli` | PipeWire CLI | `pw-cli list-objects` |
| `systemctl --user` | 用户服务管理 | `systemctl --user restart wireplumber` |

## 排查流程图

```
无声音
 ├─ 1. 检查硬件 → lspci / aplay -l
 ├─ 2. 检查驱动 → lsmod / dmesg
 ├─ 3. 检查 ALSA → amixer / speaker-test
 ├─ 4. 检查 PipeWire → systemctl / wpctl / 日志
 └─ 5. 执行修复
     ├─ 重启 WirePlumber
     ├─ 设置默认设备
     ├─ 调整音量/取消静音
     └─ 检查权限
```

## 参考资料

- [ALSA 文档](https://www.alsa-project.org/)
- [PipeWire 文档](https://pipewire.org/)
- [WirePlumber 文档](https://pipewire.pages.freedesktop.org/wireplumber/)
- [Arch Linux Wiki - PipeWire](https://wiki.archlinux.org/title/PipeWire)
