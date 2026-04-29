# 地面渲染修复指南

## 原项目地面渲染分析（基于 lorien.cfg 和 map/init.lua）

### 关键参数

| 参数 | 值 | 说明 |
|------|-----|------|
| `horizon` | 327 | 地面起始Y位置 |
| `floor.top` | tile/2.png | 地面顶部瓦片 |
| `floor.extra` | tile/3.png | 地面填充瓦片 |
| 地图宽度 | 1760 | 1280 + 480 |

### 渲染逻辑

```lua
-- 从 horizon 开始水平平铺 top 瓦片 (tile/2.png)
while (x < data.info.width) do
    -- 添加顶部瓦片
    table.insert(data.layer.floor, {sprite = top.path, x = x, y = y})
    
    -- 垂直向下填充 extra 瓦片 (tile/3.png)
    y = y + height
    while (y < data.info.height) do
        table.insert(data.layer.floor, {sprite = extra.path, x = x, y = y})
        y = y + extra.spriteData.h
    end
    
    x = x + top.spriteData.w
end
```

---

## Godot 地面场景修复

### 步骤 1：检查瓦片纹理尺寸

在 Godot 编辑器中查看：
- `res://asset/textures/map/lorien/tile/2.png` - 顶部瓦片
- `res://asset/textures/map/lorien/tile/3.png` - 填充瓦片

假设瓦片尺寸为 256x64（根据项目常见尺寸）

### 步骤 2：创建正确的 floor.tscn

编辑 `/Volumes/data/dev/code/game/DFQ-Original/dfq/scenes/floor.tscn`：

```gdscript
[gd_scene format=3]

[ext_resource type="Texture2D" path="res://asset/textures/map/lorien/tile/2.png" id="1_tile_top"]
[ext_resource type="Texture2D" path="res://asset/textures/map/lorien/tile/3.png" id="2_tile_extra"]

[sub_resource type="RectangleShape2D" id="RectangleShape2D_123"]
size = Vector2(1760, 50)

[node name="FloorRoot" type="Node2D"]
position = Vector2(0, 327)

# 顶部瓦片层（水平平铺）
[node name="TopTiles" type="Node2D" parent="."]

# 填充瓦片层（垂直向下）
[node name="ExtraTiles" type="Node2D" parent="."]

# 碰撞体
[node name="CollisionBody" type="StaticBody2D" parent="."]

[node name="CollisionShape2D" type="CollisionShape2D" parent="CollisionBody"]
shape = SubResource("RectangleShape2D_123")
```

### 步骤 3：通过脚本生成瓦片

在 `game_manager.gd` 中添加瓦片生成函数，或者创建一个独立的 `floor_generator.gd` 脚本：

```gdscript
# 脚本位置：res://scripts/floor_generator.gd
class_name FloorGenerator
extends Node

const TILE_WIDTH = 256
const TILE_HEIGHT = 64
const MAP_WIDTH = 1760
const HORIZON_Y = 327
const SCREEN_HEIGHT = 720

func generate_floor(root_node: Node2D):
    var texture_top = load("res://asset/textures/map/lorien/tile/2.png")
    var texture_extra = load("res://asset/textures/map/lorien/tile/3.png")
    
    var top_node = root_node.get_node_or_null("TopTiles")
    var extra_node = root_node.get_node_or_null("ExtraTiles")
    
    if not top_node or not extra_node:
        return
    
    # 生成顶部瓦片（水平平铺）
    var x = 0
    while x < MAP_WIDTH:
        var sprite = Sprite2D.new()
        sprite.texture = texture_top
        sprite.position = Vector2(x, 0)
        top_node.add_child(sprite)
        x += TILE_WIDTH
    
    # 生成填充瓦片（每个X列，垂直向下）
    x = 0
    while x < MAP_WIDTH:
        var y = TILE_HEIGHT  # 从顶部瓦片下面开始
        while y < SCREEN_HEIGHT - HORIZON_Y:
            var sprite = Sprite2D.new()
            sprite.texture = texture_extra
            sprite.position = Vector2(x, y)
            extra_node.add_child(sprite)
            y += TILE_HEIGHT
        x += TILE_WIDTH
    
    print("Floor generated")
```

### 步骤 4：在 game_manager.gd 中集成

在 `setup_floor()` 中调用：

```gdscript
func setup_floor():
    var floor_scene = load("res://scenes/floor.tscn")
    if floor_scene:
        var floor_root = floor_scene.instantiate()
        floor_root.position = Vector2(0, 327)  # horizon = 327
        add_child(floor_root)
        
        # 调用瓦片生成
        var generator = load("res://scripts/floor_generator.gd").new()
        generator.generate_floor(floor_root)
        
        print("Floor created with tiles")
    else:
        print("Error: Could not load floor.tscn")
```

### 简化方案：使用 TextureRect + Repeat

如果不想写生成脚本，可以使用 Godot 的 TextureRect 带重复模式：

```gdscript
[gd_scene format=3]

[ext_resource type="Texture2D" path="res://asset/textures/map/lorien/tile/2.png" id="1_tile_top"]
[ext_resource type="Texture2D" path="res://asset/textures/map/lorien/tile/3.png" id="2_tile_extra"]

[sub_resource type="RectangleShape2D" id="RectangleShape2D_123"]
size = Vector2(1760, 50)

[node name="FloorRoot" type="Node2D"]
position = Vector2(0, 327)

# 顶部瓦片（水平重复）
[node name="TopRect" type="TextureRect" parent="."]
texture = ExtResource("1_tile_top")
expand_mode = 1  # Keep Aspect Ratio
stretch_mode = 5  # Tile
size = Vector2(1760, 64)

# 填充瓦片（水平重复，垂直重复）
[node name="ExtraRect" type="TextureRect" parent="."]
texture = ExtResource("2_tile_extra")
expand_mode = 1
stretch_mode = 5
size = Vector2(1760, 720 - 327 - 64)
position = Vector2(0, 64)

# 碰撞体
[node name="CollisionBody" type="StaticBody2D" parent="."]

[node name="CollisionShape2D" type="CollisionShape2D" parent="CollisionBody"]
shape = SubResource("RectangleShape2D_123")
position = Vector2(0, 0)
```

---

## 验证清单

- [ ] 地面起始位置在 Y=327
- [ ] 顶部瓦片水平平铺覆盖 1760 宽度
- [ ] 填充瓦片从 Y=64 开始向下铺满
- [ ] 碰撞体覆盖整个地面宽度
- [ ] 角色正确站在地面上

---

*创建日期: 2026-04-29*
