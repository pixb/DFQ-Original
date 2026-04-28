# Phase 22: 修复画面显示

## 问题描述

运行游戏后只能看到蓝色背景，看不到玩家、敌人和地板的任何可见元素。

## 根本原因

1. **缺少相机**：游戏场景没有 Camera2D 节点
2. **缺少背景色**：渲染配置没有设置默认背景
3. **缺少碰撞形状**：CollisionShape2D 没有设置 shape 资源
4. **Sprite2D 无纹理**：使用 PlaceholderTexture2D 不可见

## 修复内容

### 1. 添加相机

在 `scripts/game_manager.gd` 的 `_ready()` 中创建相机：

```gdscript
func setup_camera():
	var camera = Camera2D.new()
	camera.position = Vector2(640, 360)
	add_child(camera)
	print("Camera created at: ", camera.position)
```

### 2. 设置背景色

修改 `project.godot` 的渲染配置：

```ini
[rendering]

environment/defaults/default_clear_color = Color(0.2, 0.2, 0.3, 1)
rendering_device/driver.windows="d3d12"
renderer/rendering_method="gl_compatibility"
renderer/rendering_method.mobile="gl_compatibility"
```

### 3. 添加窗口设置

```ini
[display]

window/size/viewport_width=1280
window/size/viewport_height=720
window/size/resizable=true
window/size/centered=true
window/stretch/mode="viewport"
window/stretch/aspect="expand"
```

### 4. 修复碰撞形状

#### 玩家场景

```gdscript
[sub_resource type="CapsuleShape2D" id="CapsuleShape2D_player"]
radius = 16.0
height = 48.0
```

#### 地板场景

```gdscript
[sub_resource type="RectangleShape2D" id="RectangleShape2D_floor"]
size = Vector2(40, 1)
```

### 5. 添加调试矩形（临时方案）

由于 PlaceholderTexture2D 不可见，添加 `ColorRect` 作为替代：

#### 玩家

```gdscript
[node name="DebugRect" type="ColorRect" parent="."]
anchors_preset = 8
anchor_left = 0.5
anchor_top = 0.5
anchor_right = 0.5
anchor_bottom = 0.5
grow_horizontal = 2
grow_vertical = 2
offset_left = -48.0
offset_top = -36.0
offset_right = 48.0
offset_bottom = 36.0
color = Color(1, 1, 0, 1)  # 黄色
```

#### 敌人

```gdscript
[node name="DebugRect" type="ColorRect" parent="."]
color = Color(1, 0, 0, 1)  # 红色
```

#### 地板

```gdscript
[node name="DebugRect" type="ColorRect" parent="."]
color = Color(0.5, 0.5, 0.5, 1)  # 灰色
```

## 测试结果

✅ 蓝色背景显示
✅ 黄色玩家矩形可见
✅ 3个红色敌人矩形可见
✅ 灰色地板矩形可见
✅ 玩家可以移动、跳跃、攻击
✅ 敌人有AI行为

## 后续优化

1. **替换为真实精灵图**：将 ColorRect 替换为 Sprite2D + texture
2. **连接血条**：将血条UI连接到玩家伤害系统
3. **添加背景音乐**：集成 BGM 系统
4. **优化视觉**：调整精灵大小、添加动画

## 相关文件

- `scripts/game_manager.gd` - 添加相机
- `project.godot` - 窗口和背景设置
- `scenes/player.tscn` - 添加调试矩形和碰撞形状
- `scenes/enemy.tscn` - 添加调试矩形和碰撞形状
- `scenes/floor.tscn` - 添加调试矩形和碰撞形状

---

*Updated: 2026-04-28*
