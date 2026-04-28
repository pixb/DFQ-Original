# Phase 13: UI System Migration

## Overview

迁移 UI 系统，包括标签、面板、按钮和 UI 管理器。

## LÖVE Reference Files

- `source/graphics/drawable/label.lua` - Label drawable

## Godot Implementation

### ui_label.gd

UI 标签控件:
- text: String - 显示文本
- font: Font - 字体
- font_size: int - 字体大小
- align: int - 水平对齐 (0=left, 1=center, 2=right)
- valign: int - 垂直对齐 (0=top, 1=center, 2=bottom)
- color: Color - 文本颜色

方法:
- set_text(new_text) - 设置文本
- get_text() - 获取文本
- set_font(new_font) - 设置字体
- set_font_size(size) - 设置字号
- set_horizontal_alignment(alignment) - 设置水平对齐
- set_vertical_alignment(alignment) - 设置垂直对齐
- set_color(new_color) - 设置颜色
- get_text_width() - 获取文本宽度
- get_text_height() - 获取文本高度

### ui_panel.gd

UI 面板容器:
- _layout: VBoxContainer - 内部布局

方法:
- add_child_control(control) - 添加控件
- remove_child_control(control) - 移除控件
- clear() - 清空
- get_layout() - 获取布局容器

### ui_button.gd

UI 按钮:
- click_sound: AudioStream - 点击音效
- hover_sound: AudioStream - 悬停音效

方法:
- set_text(text) - 设置文本
- set_normal_texture(texture) - 设置普通纹理
- set_hover_texture(texture) - 设置悬停纹理
- set_pressed_texture(texture) - 设置按下纹理

### ui_system.gd

UI 系统管理器 (CanvasLayer):
- _screens: Dictionary - 已注册界面
- _current_screen: Control - 当前显示界面
- _ui_layer: int - UI 层

方法:
- show_screen(screen_name) - 显示界面
- hide_screen() - 隐藏界面
- register_screen(name, screen) - 注册界面
- unregister_screen(name) - 注销界面
- get_screen(name) - 获取界面
- get_current_screen() - 获取当前界面
- set_ui_layer(layer) - 设置层
- show_dialog(title, message, buttons, callback) - 显示对话框
- show_toast(message, duration) - 显示提示

## Godot UI Approach

- 使用 Godot Control 节点系统
- CanvasLayer 用于 UI 层管理
- 构建在 Godot GUI 系统之上
- 支持屏幕管理和对话框

## Testing

- UI container test: CoreUtils.Container works
- Phase 13 Complete