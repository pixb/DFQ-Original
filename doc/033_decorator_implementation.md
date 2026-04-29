# Phase 32: 场景装饰系统 - 实现指南

## 概述

本文档详细说明如何实现场景装饰系统，包括树木、石头、花草等装饰元素的添加和视差滚动。

## 资源复制步骤

由于权限限制，需要手动复制以下资源到Godot项目：

### 1. 创建目录结构

在终端执行：

```bash
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/tree
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/stone
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/flower
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/grass
```

### 2. 复制资源文件

```bash
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/tree/*.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/tree/
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/stone/*.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/stone/
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/flower/*.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/flower/
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/grass/*.png /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/grass/
```

## 代码实现

### 1. 创建装饰管理器脚本

在 `scripts/decorator_manager.gd` 中创建：

```gdscript
class_name DecoratorManager
extends Node2D

var decorations: Array = []
var camera_reference: Camera2D = null

func _ready():
	print("DecoratorManager initialized")

func setup_decorations(cam: Camera2D):
	camera_reference = cam

	var dec_layer = Node2D.new()
	dec_layer.name = "DecorationLayer"
	add_child(dec_layer)

	add_decoration(dec_layer, "res://asset/map/lorien/tree/0.png", Vector2(200, 380), 0.4)
	add_decoration(dec_layer, "res://asset/map/lorien/tree/1.png", Vector2(500, 360), 0.4)
	add_decoration(dec_layer, "res://asset/map/lorien/tree/2.png", Vector2(900, 370), 0.4)
	add_decoration(dec_layer, "res://asset/map/lorien/tree/3.png", Vector2(1200, 350), 0.4)

	add_decoration(dec_layer, "res://asset/map/lorien/stone/0.png", Vector2(300, 450), 0.6)
	add_decoration(dec_layer, "res://asset/map/lorien/stone/1.png", Vector2(700, 460), 0.6)
	add_decoration(dec_layer, "res://asset/map/lorien/stone/2.png", Vector2(1100, 440), 0.6)

	print("Decorations setup complete: ", decorations.size(), " decorations loaded")

func add_decoration(parent: Node2D, texture_path: String, position: Vector2, rate: float):
	var sprite = Sprite2D.new()
	var texture = load(texture_path)

	if texture:
		sprite.texture = texture
		sprite.position = position
		sprite.set_meta("original_position", position)
		sprite.set_meta("parallax_rate", rate)
		parent.add_child(sprite)
		decorations.append(sprite)
		print("Added decoration: ", texture_path)

func _process(delta):
	if camera_reference and not decorations.is_empty():
		update_decorations()

func update_decorations():
	var camera_pos = camera_reference.position

	for dec in decorations:
		if is_instance_valid(dec):
			var original_pos: Vector2 = dec.get_meta("original_position", dec.position)
			var rate: float = dec.get_meta("parallax_rate", 0.5)

			dec.position = Vector2(
				original_pos.x - camera_pos.x * rate,
				original_pos.y
			)
```

### 2. 创建装饰层场景

在 `scenes/decoration_layer.tscn` 中创建：

```gd_scene
[gd_scene format=3]

[node name="DecorationLayer" type="Node2D"]
script = ExtResource("1_decorator")

[ext_resource type="Script" path="res://scripts/decorator_manager.gd" id="1_decorator"]
```

### 3. 更新game_manager.gd

在 `game_manager.gd` 的 `_ready()` 函数中添加：

```gdscript
var decorator_manager: DecoratorManager

func _ready():
	print("Game started!")
	setup_camera()
	setup_background()
	setup_decorations()  # 添加这行
	setup_floor()
	# ... 其他setup调用
```

添加新的setup函数：

```gdscript
func setup_decorations():
	decorator_manager = DecoratorManager.new()
	decorator_manager.name = "DecoratorManager"
	add_child(decorator_manager)
	decorator_manager.setup_decorations(camera)
	print("Decoration system ready")
```

## 装饰布局配置

### 远景装饰（rate: 0.3-0.4）
- 大型树木
- 山脉岩石

### 中景装饰（rate: 0.5-0.6）
- 中型树木
- 石头
- 灌木

### 近景装饰（rate: 0.7-0.8）
- 小型花草
- 地面石子
- 地面装饰

## 装饰类型配置

| 类型 | 路径 | 建议位置Y | 视差率 |
|------|------|----------|--------|
| 大树 | tree/0.png, tree/1.png | 350-400 | 0.4 |
| 中树 | tree/2.png, tree/3.png | 380-420 | 0.5 |
| 小树 | tree/4.png, tree/5.png | 400-450 | 0.6 |
| 大石 | stone/0.png, stone/1.png | 440-470 | 0.6 |
| 小石 | stone/2.png, stone/3.png | 450-480 | 0.7 |
| 花 | flower/*.png | 460-500 | 0.8 |
| 草 | grass/*.png | 470-510 | 0.8 |

## 测试步骤

1. 确保资源已复制到正确目录
2. 运行项目
3. 检查控制台输出：
   - "DecoratorManager initialized"
   - "Decorations setup complete: X decorations loaded"
   - "Decoration system ready"
4. 移动玩家，观察装饰元素的视差效果

## 性能优化建议

1. **对象池**：如果装饰元素很多，使用对象池管理
2. **LOD**：远离相机的装饰使用低分辨率纹理
3. **批处理**：使用 TextureRect 而非多个 Sprite2D
4. **可见性**：只加载屏幕范围内的装饰

## 常见问题

### Q: 装饰不显示
- 检查资源路径是否正确
- 检查纹理是否成功加载
- 检查装饰的Y位置是否在屏幕范围内

### Q: 视差效果不明显
- 调整各装饰的 parallax_rate
- 确保相机在移动
- 检查 update_decorations 是否被调用

### Q: 性能问题
- 减少装饰数量
- 使用对象池
- 优化装饰的更新频率

---

*创建日期: 2026-04-29*
