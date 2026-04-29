# Phase 36: 完整地图系统

## 概述

本阶段实现Godot的完整地图系统，包括TileMap、碰撞检测、区域切换等功能。

## 地图系统架构

```
MapManager (Autoload)
├── current_map: TileMap
├── map_data: Dictionary
├── regions: Array
├── load_map()
├── save_map()
├── change_region()
└── get_tile_info()
```

## 实现方案

### 1. 地图管理器

```gdscript
# scripts/map_manager.gd
class_name MapManager
extends Node

enum MapLayer { GROUND, COLLISION, DECORATION, OBJECTS }

var current_map: TileMap
var map_data: Dictionary = {}
var regions: Array = []
var camera_bounds: Rect2

func _ready():
    load_default_map()
    print("MapManager initialized")

func load_default_map():
    var map_scene = load("res://scenes/maps/lorien.tscn")
    if map_scene:
        current_map = map_scene.instantiate()
        get_tree().current_scene.add_child(current_map)
        print("Map loaded: lorien")
    
    load_regions()
    setup_camera_bounds()

func load_regions():
    regions = [
        {"name": "forest", "bounds": Rect2(0, 0, 1000, 720)},
        {"name": "gate", "bounds": Rect2(1000, 0, 500, 720)},
        {"name": "cave", "bounds": Rect2(1500, 0, 800, 720)}
    ]
    print("Loaded ", regions.size(), " regions")

func setup_camera_bounds():
    if current_map:
        var map_size = current_map.get_used_rect().size * current_map.tile_size
        camera_bounds = Rect2(0, 0, map_size.x, map_size.y)
        print("Camera bounds set: ", camera_bounds)

func change_region(region_name: String):
    for region in regions:
        if region["name"] == region_name:
            var camera = get_node_or_null("/root/GameManager/Camera2D")
            if camera:
                camera.position = region["bounds"].position + region["bounds"].size / 2
            print("Changed to region: ", region_name)
            return
    print("Region not found: ", region_name)

func get_current_region(position: Vector2) -> String:
    for region in regions:
        if region["bounds"].has_point(position):
            return region["name"]
    return "unknown"

func get_tile_at(position: Vector2) -> int:
    if not current_map:
        return -1
    
    var tile_pos = current_map.world_to_map(position)
    return current_map.get_cell(MapLayer.COLLISION, tile_pos)

func is_walkable(position: Vector2) -> bool:
    var tile_id = get_tile_at(position)
    return tile_id == 0
```

### 2. TileMap场景创建

```gd_scene
[gd_scene format=3]

[node name="LorienMap" type="TileMap"]
tile_set = ExtResource("1_tileset")

[node name="GroundLayer" type="TileMap" parent="."]
tile_set = ExtResource("1_tileset")
layer_name = "ground"

[node name="CollisionLayer" type="TileMap" parent="."]
tile_set = ExtResource("1_tileset")
layer_name = "collision"

[node name="DecorationLayer" type="TileMap" parent="."]
tile_set = ExtResource("1_tileset")
layer_name = "decoration"

[ext_resource type="TileSet" path="res://asset/tilesets/lorien_tileset.tres" id="1_tileset"]
```

### 3. 区域检测系统

```gdscript
# scripts/region_detector.gd
class_name RegionDetector
extends Area2D

var region_name: String
var on_enter_callback: Callable
var on_exit_callback: Callable

func _ready():
    body_entered.connect(_on_body_entered)
    body_exited.connect(_on_body_exited)

func _on_body_entered(body: Node2D):
    if body.name == "Player":
        print("Player entered region: ", region_name)
        if on_enter_callback:
            on_enter_callback.call(region_name)

func _on_body_exited(body: Node2D):
    if body.name == "Player":
        print("Player exited region: ", region_name)
        if on_exit_callback:
            on_exit_callback.call(region_name)
```

### 4. 地图加载系统

```gdscript
# scripts/map_loader.gd
class_name MapLoader
extends Node

var maps: Dictionary = {
    "lorien": "res://scenes/maps/lorien.tscn",
    "cave": "res://scenes/maps/cave.tscn",
    "castle": "res://scenes/maps/castle.tscn"
}

func load_map(map_name: String):
    if not maps.has(map_name):
        print("Map not found: ", map_name)
        return null
    
    var map_scene = load(maps[map_name])
    if map_scene:
        return map_scene.instantiate()
    
    return null

func async_load_map(map_name: String) -> PackedScene:
    var map_path = maps.get(map_name)
    if not map_path:
        return null
    
    await get_tree().create_timer(0.1).timeout
    return load(map_path)

func get_all_map_names() -> Array:
    return maps.keys()
```

## 集成到游戏管理器

```gdscript
# game_manager.gd
var map_manager: MapManager

func _ready():
    # ... 其他初始化 ...
    setup_map_manager()

func setup_map_manager():
    map_manager = MapManager.new()
    map_manager.name = "MapManager"
    add_child(map_manager)
    print("MapManager setup")

func on_player_move(position: Vector2):
    var region = map_manager.get_current_region(position)
    if region != current_region:
        current_region = region
        on_region_change(region)

func on_region_change(region_name: String):
    print("Entered region: ", region_name)
    
    # 根据区域切换天气
    match region_name:
        "forest":
            weather_manager.set_weather(WeatherManager.WeatherType.SUNNY)
        "cave":
            weather_manager.set_weather(WeatherManager.WeatherType.FOG)
        "gate":
            weather_manager.set_weather(WeatherManager.WeatherType.STORM)
```

## 碰撞检测优化

```gdscript
# scripts/collision_optimization.gd
class_name CollisionOptimizer
extends Node

var collision_cells: Dictionary = {}

func optimize_collision(tile_map: TileMap):
    collision_cells.clear()
    
    var used_rect = tile_map.get_used_rect()
    for x in range(used_rect.position.x, used_rect.position.x + used_rect.size.x):
        for y in range(used_rect.position.y, used_rect.position.y + used_rect.size.y):
            var tile_id = tile_map.get_cell(0, Vector2i(x, y))
            if tile_id != 0:
                collision_cells[Vector2i(x, y)] = tile_id
    
    print("Collision optimized: ", collision_cells.size(), " cells")

func is_collision(position: Vector2, tile_size: Vector2) -> bool:
    var tile_pos = Vector2i(
        int(position.x / tile_size.x),
        int(position.y / tile_size.y)
    )
    return collision_cells.has(tile_pos)
```

## 地图编辑器集成

### 在Godot编辑器中设置TileSet

1. 创建TileSet资源
2. 添加纹理图集
3. 设置碰撞形状
4. 分配tile ID
5. 设置动画帧

### 地图数据格式

```json
{
  "name": "lorien",
  "width": 200,
  "height": 30,
  "tile_size": 32,
  "layers": [
    {
      "name": "ground",
      "data": [...]
    },
    {
      "name": "collision",
      "data": [...]
    },
    {
      "name": "decoration",
      "data": [...]
    }
  ],
  "regions": [
    {"name": "forest", "x": 0, "y": 0, "width": 100, "height": 30},
    {"name": "gate", "x": 100, "y": 0, "width": 50, "height": 30}
  ],
  "spawn_points": {
    "player": {"x": 100, "y": 10},
    "enemies": [{"x": 150, "y": 10}, {"x": 160, "y": 10}]
  }
}
```

## 测试验证

### 测试用例

| 测试项 | 操作 | 预期结果 |
|--------|------|----------|
| 地图加载 | 启动游戏 | 地图正常显示 |
| 碰撞检测 | 玩家移动 | 无法穿过障碍物 |
| 区域切换 | 玩家移动 | 区域变化时触发事件 |
| 天气切换 | 进入不同区域 | 天气自动切换 |

## 性能优化

### 1. 视锥剔除

```gdscript
func cull_tiles(camera_rect: Rect2):
    var tile_size = current_map.tile_size
    var start_tile = current_map.world_to_map(camera_rect.position)
    var end_tile = current_map.world_to_map(camera_rect.end)
    
    for layer in current_map.get_layers():
        current_map.set_layer_visible(layer, true)
        # 只渲染可见区域
```

### 2. 层级管理

使用不同的TileMap节点管理不同层级，便于渲染优化。

### 3. 异步加载

```gdscript
func _ready():
    await async_load_map("lorien")
```

## 后续改进

- ✅ Phase 37: 游戏优化与发布

## 相关文档

- map_manager_gd.md - 地图管理器代码模板
- region_detector_gd.md - 区域检测器代码模板
- map_loader_gd.md - 地图加载器代码模板
- collision_optimization_gd.md - 碰撞优化代码模板
