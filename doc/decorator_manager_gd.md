# 装饰管理器脚本

## 功能

管理场景中的装饰元素，包括树木、石头、花草等，并支持视差滚动效果。

## 代码

```gdscript
# scripts/decorator_manager.gd
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
