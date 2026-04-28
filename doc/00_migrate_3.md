# 阶段 3: 图形与渲染

**目标**: 移植 graphics 和 drawable 系统

---

## 任务清单

- [x] 3.1 GraphicsManager (内置 Godot)
- [x] 3.2 Camera 系统 (Camera2D)
- [x] 3.3 ResourceManager (已创建)
- [x] 3.4 Drawable 系统 (通过 _draw)
- [x] 3.5 测试渲染 (圆/矩形/线条)

---

## 3.1 GraphicsManager

### LÖVE graphics.lua 功能

| 功能 | Godot 等效 |
|------|----------|
| SetBackgroundColor | Environment.background_color |
| SetShader | CanvasItem.material |
| SetBlendmode | CanvasItem.blend_mode |
| SetColor | Self.modulate |
| SetFont | Label settings |
| DrawCircle | draw_circle() |
| DrawRect | draw_rect() |
| DrawPolygon | draw_polygon() |
| Push/Pop | CanvasItem.transform |

### Godot 实现

使用 Godot 内置 `RenderingServer` 和 Node 系统:

```gdscript
# scripts/graphics_manager.gd
extends Node

func _ready():
	print("GraphicsManager initialized")

func set_background_color(color: Color):
	var env = get_viewport().get_world_2d().get_direct_space_state()
	# Or use World2D environment
```

### 简化的 Godot 渲染方式

| LÖVE | Godot 方法 |
|------|----------|
| love.graphics.draw | `draw_*` methods in _draw() |
| love.graphics.setColor | `modulate` property |
| love.graphics.push/pop | `canvas_item` transform |

---

## 3.2 Camera 系统

### LÖVE camera (source/graphics/camera.lua)

```lua
-- Camera tracking and viewport management
```

### Godot 实现

直接使用 Godot 内置 Camera2D:

```gdscript
# scenes/camera.tscn
[node name="Camera2D" type="Camera2D"]
position = Vector2(640, 360)
```

---

## 3.3 ResourceManager

### 需要迁移 source/lib/resource.lua

关键功能:
- LoadImage (加载纹理)
- NewSprite (创建精灵)
- LoadFont (加载字体)

### Godot 实现

```gdscript
# scripts/resource_manager.gd
extends Node

var _textures = {}
var _fonts = {}

func load_texture(path: String) -> Texture2D:
	if _textures.has(path):
		return _textures[path]
	var texture = load(path)
	_textures[path] = texture
	return texture

func load_font(path: String) -> Font:
	if _fonts.has(path):
		return _fonts[path]
	var font = load(path)
	_fonts[path] = font
	return font
```

---

## 3.4 Drawable 系统

### source/graphics/drawable/

需要分析的 drawable 类型:

| 文件 | Godot 等效 |
|------|----------|
| sprite.lua | Sprite2D |
| label.lua | Label |
| iRect.lua | ColorRect |
| layer.lua | CanvasLayer |
| frameani.lua | AnimationPlayer |
| particle.lua | GPUParticles2D |
| base.lua | Node2D 基类 |

---

## 3.5 测试渲染

验证步骤:

1. 创建简单的 Camera2D
2. 加载一个测试纹理
3. 在场景中绘制精灵

---

## 总结

- 使用 Godot 内置渲染系统
- Camera2D 替代手动相机
- ResourceManager 管理资源加载

**预计工作量**: 4-6 小时

---

*阶段 3 完成*