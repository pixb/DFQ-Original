# 完整视差滚动系统

## 概述

在 Phase 30 的基础上，实现真正的视差滚动效果，使用原项目的背景纹理，并根据相机位置动态更新背景层。

## 背景资源

从原项目复制背景纹理到 Godot 项目：

- `asset/map/lorien/far.png` - 远景背景（视差率 0.3）
- `asset/map/lorien/near.png` - 近景背景（视差率 0.7）

## 实现方案

### 1. 视差背景场景 (`scenes/parallax_background.tscn`)

使用 `TextureRect` 替代 `ColorRect`，便于使用真实纹理：

```gdscript
[gd_scene format=3]

[node name="ParallaxBackground" type="CanvasLayer"]
layer = -1

[node name="SkyLayer" type="ColorRect" parent="."]
anchors_preset = 15
anchor_right = 1.0
anchor_bottom = 1.0
grow_horizontal = 2
grow_vertical = 2
color = Color(0.529, 0.808, 0.922, 1)

[node name="FarBackground" type="TextureRect" parent="."]
anchors_preset = 0
offset_left = 0.0
offset_top = 0.0
offset_right = 2560.0
offset_bottom = 720.0
grow_horizontal = 2
grow_vertical = 2
stretch_mode = 2

[node name="NearBackground" type="TextureRect" parent="."]
anchors_preset = 0
offset_left = 0.0
offset_top = 0.0
offset_right = 2560.0
offset_bottom = 720.0
grow_horizontal = 2
grow_vertical = 2
stretch_mode = 2
```

### 2. 游戏管理器中的视差逻辑 (`scripts/game_manager.gd`)

添加相机跟踪和背景更新：

```gdscript
var camera: Camera2D
var far_background: TextureRect
var near_background: TextureRect

func _process(delta):
    if current_state == GameState.PLAYING and player:
        update_parallax()

func update_parallax():
    if far_background and near_background:
        var camera_pos = camera.position
        far_background.position = Vector2(-camera_pos.x * 0.3, -camera_pos.y * 0.3)
        near_background.position = Vector2(-camera_pos.x * 0.7, -camera_pos.y * 0.7)

func setup_background():
    var bg_scene = load("res://scenes/parallax_background.tscn")
    var bg = bg_scene.instantiate()
    add_child(bg)
    
    far_background = bg.get_node("FarBackground")
    near_background = bg.get_node("NearBackground")
    
    var far_texture = load("res://asset/map/lorien/far.png")
    var near_texture = load("res://asset/map/lorien/near.png")
    
    if far_texture:
        far_background.texture = far_texture
        far_background.stretch_mode = TextureRect.STRETCH_TILE
    if near_texture:
        near_background.texture = near_texture
        near_background.stretch_mode = TextureRect.STRETCH_TILE
    
    print("Parallax background loaded with textures")
```

### 3. 背景层基类 (`scripts/background_layer.gd`) 备选方案

如果需要更高级的视差控制（如重复纹理）：

```gdscript
class_name BackgroundLayer
extends Node2D

var texture: Texture2D = null
var parallax_rate: float = 0.3
var scroll_offset: Vector2 = Vector2.ZERO
var is_repeating: bool = true

func set_texture(new_texture: Texture2D) -> void:
    texture = new_texture
    queue_redraw()

func update_scroll(camera_position: Vector2) -> void:
    scroll_offset = camera_position * parallax_rate
    queue_redraw()

func _draw() -> void:
    if not texture:
        return
    
    var draw_x: float = fmod(scroll_offset.x, texture.get_size().x)
    if draw_x > 0:
        draw_x -= texture.get_size().x
    
    var draw_count: int = ceil(2560.0 / texture.get_size().x) + 2
    
    for i in range(draw_count):
        var pos: Vector2 = Vector2(draw_x + i * texture.get_size().x, 0)
        draw_texture(texture, pos)
```

## 实现步骤

### 步骤 1: 复制资源文件

在终端执行：

```bash
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/far.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/near.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/
```

### 步骤 2: 更新视差背景场景

将 `scenes/parallax_background.tscn` 更新为使用 `TextureRect` 的版本。

### 步骤 3: 完善 game_manager

在 `scripts/game_manager.gd` 中添加视差滚动逻辑，包括：
- 相机引用
- 背景层引用
- `_process` 函数更新视差
- `update_parallax` 函数处理偏移计算
- `setup_background` 函数加载纹理

## 视差原理

视差效果通过不同层使用不同的移动速率实现：

- 天空层（SkyLayer）: 固定不动（rate = 0.0）
- 远景层（FarBackground）: 慢速移动（rate = 0.3）
- 近景层（NearBackground）: 快速移动（rate = 0.7）

当玩家移动时，相机跟随玩家，各层背景根据相机位置和自身速率计算偏移：

```
layer_position = -camera_position * parallax_rate
```

负号确保背景移动方向与玩家相反，创造深度感。

## 纹理重复

使用 `TextureRect.STRETCH_TILE` 实现纹理无缝重复，确保背景在任何相机位置下都能完整显示。

## 注意事项

1. **资源加载**: 确保背景纹理路径正确
2. **相机跟踪**: 确保相机正确跟随玩家
3. **性能优化**: 视差计算在 `_process` 中执行，确保效率
4. **深度层级**: 使用 `CanvasLayer.layer` 确保正确的绘制顺序

## 后续优化

- 使用 `ParallaxBackground` 和 `ParallaxLayer` 节点替代手动计算
- 添加更多装饰元素（树木、石头等）
- 实现平滑滚动和缓动效果
- 根据玩家移动速度调整视差速率

## 测试

运行项目后应该看到：
1. 蓝色天空背景
2. 远景层纹理（far.png）
3. 近景层纹理（near.png）
4. 当玩家移动时，各层以不同速度滚动
