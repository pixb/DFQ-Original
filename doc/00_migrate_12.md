# Phase 12: Map and Scene System Migration

## Overview

迁移地图和场景系统，包括地图管理、相机和背景层。

## LÖVE Reference Files

- `source/map/init.lua` - MAP 主类
- `source/map/camera.lua` - Map 相机
- `source/map/background.lua` - 背景层
- `source/graphics/camera.lua` - 图形相机基类

## Godot Implementation

### camera.gd

地图跟随相机:
- position: Vector2 - 当前位置
- current: Vector2 - 当前平滑位置
- target: Node - 跟随目标
- target_position: Vector2 - 目标位置
- follow_speed: float - 跟随速度
- follow_time: float - 跟随时间
- world_scale/target_scale: Vector2 - 缩放
- world_bounds: Rect2 - 世界边界
- shake_offset/shake_timer/shake_intensity - 震动效果

方法:
- set_target(new_target, time) - 设置跟随目标
- set_world_bounds(x, y, width, height) - 设置边界
- set_position(x, y, adjust) - 设置位置
- shake(time, xa, xb, ya, yb) - 震动效果
- set_scale(x, y, time) - 缩放
- get_shift() - 获取偏移
- is_moving() - 是否移动中
- is_scaling() - 是否缩放中
- apply() - 应用震动偏移

### map_system.gd

地图系统:
- camera: GameCamera - 相机
- matrix_group: Dictionary - 矩阵组
- info: Dictionary - 地图信息
- values: Dictionary - BGM/BGS 配置
- load_process: int - 加载进度
- is_paused: bool - 暂停状态

方法:
- init() - 初始化
- update(dt) - 更新
- load(path, adjust) - 加载地图
- make(path, entry) - 生成地图
- get_matrix(key) - 获取矩阵
- get_load_process() - 获取加载进度
- set_paused(paused) - 设置暂停

### background_layer.gd

背景层:
- sprite_path: String - 精灵路径
- position: Vector2 - 位置
- parallax: float - 视差

方法:
- set_image(path) - 设置图片
- draw() - 绘制

## Features

- 相机跟随目标
- 相机震动效果
- 视差背景
- 地图加载流程
- 矩阵系统

## Testing

- Map container test: CoreUtils.Container works
- Phase 12 Complete