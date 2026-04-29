# 装饰层场景

## 场景文件

```gd_scene
[gd_scene format=3]

[node name="DecorationLayer" type="Node2D"]
script = ExtResource("1_decorator")

[ext_resource type="Script" path="res://scripts/decorator_manager.gd" id="1_decorator"]
```

## 使用方法

1. 将此场景添加到 `game_manager.gd` 中
2. 在 `setup_background()` 之后调用 `setup_decorations(camera)`
3. 装饰元素将自动参与视差滚动

## 示例

在 `game_manager.gd` 中添加：

```gdscript
var decorator_manager: DecoratorManager

func setup_background():
	# ... 现有的背景设置 ...
	
	decorator_manager = DecoratorManager.new()
	decorator_manager.name = "DecoratorManager"
	add_child(decorator_manager)
	decorator_manager.setup_decorations(camera)
```
