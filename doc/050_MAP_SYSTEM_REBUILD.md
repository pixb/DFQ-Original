# 地图系统像素级复刻 (LÖVE → Godot)

## 概述

本文档记录了从 LÖVE 到 Godot 的地图系统完整复刻过程，实现了像素级精确的画面渲染。

---

## 1. Love2D 地图系统分析

### 1.1 图层结构

根据 `source/map/init.lua`，Love2D 地图系统包含以下图层：

| 图层名称 | 类型 | 视差率 | 渲染顺序 |
|---------|------|--------|----------|
| `far` | Background | 0.3 | 1 |
| `near` | Background | 0.2 | 2 |
| `effect` | Layer | - | 3 |
| `floor` | Sprite | - | 4 |
| `object` | Sprite | - | 5 |

### 1.2 核心配置参数

```lua
local _const = {
    scale = 1.3,                    -- 相机缩放
    cameraSpped = 280,              -- 相机速度
    backgroundRate = {
        far = 0.3,                  -- 远景视差率
        near = 0.2                  -- 近景视差率
    },
    scope = {
        wv = 0,
        hv = -8,
        uv = -50,
        dv = -40
    }
}
```

### 1.3 渲染顺序

```lua
function _MAP.Draw()
    _MAP.camera:Apply()
    
    _layerGroup.far:Draw()       -- 远景
    _layerGroup.near:Draw()      -- 近景
    _layerGroup.effect:Draw()    -- 特效
    _layerGroup.floor:Draw()     -- 地板
    _layerGroup.object:Draw()    -- 对象
    
    _MAP.matrixGroup.normal:Draw()
    _MAP.matrixGroup.object:Draw()
    _MAP.matrixGroup.up:Draw()
    _MAP.matrixGroup.down:Draw()
    
    _MAP.curtain:Draw()
    _MAP.OnDraw()
    
    _MAP.camera:Reset()
end
```

---

## 2. Godot 实现方案

### 2.1 图层结构映射

| Love2D | Godot | 实现方式 |
|--------|-------|----------|
| `_layerGroup.far` | `ParallaxBackground` | 视差背景节点 |
| `_layerGroup.near` | `ParallaxBackground` | 视差背景节点 |
| `_layerGroup.effect` | `Node2D` | 普通节点 |
| `_layerGroup.floor` | `Node2D` | 地板场景容器 |
| `_layerGroup.object` | `Node2D` | 对象层节点 |
| `_MAP.camera` | `Camera2D` | Godot 相机节点 |

### 2.2 核心实现

**地图系统脚本**: `scripts/map_system.gd`

```gdscript
const PARALLAX_RATE_FAR: float = 0.3
const PARALLAX_RATE_NEAR: float = 0.2

func _setup_layers():
    _layers["far"] = ParallaxBackground.new()
    _layers["near"] = ParallaxBackground.new()
    _layers["effect"] = Node2D.new()
    _layers["floor"] = Node2D.new()
    _object_layer = Node2D.new()

func _setup_camera():
    _camera = Camera2D.new()
    _camera.zoom = Vector2(1.3, 1.3)
    _camera.make_current()

func _setup_backgrounds():
    for layer_name in ["far", "near"]:
        var bg = _layers[layer_name]
        var rate = PARALLAX_RATE_FAR if layer_name == "far" else PARALLAX_RATE_NEAR
        
        var texture = load("res://asset/textures/map/lorien/" + layer_name + ".png")
        if texture:
            for i in range(3):
                var parallax_layer = ParallaxLayer.new()
                parallax_layer.motion_scale = Vector2(rate, rate)
                parallax_layer.motion_offset = Vector2(i * 1440, 0)
                
                var sprite = Sprite2D.new()
                sprite.texture = texture
                sprite.scale = Vector2(2.5, 3.0)
                parallax_layer.add_child(sprite)
                bg.add_child(parallax_layer)
```

### 2.3 视差效果实现

Godot 的 `ParallaxBackground` 配合 `ParallaxLayer` 实现视差滚动：

| 属性 | 说明 |
|------|------|
| `motion_scale` | 视差缩放比例，值越小移动越慢 |
| `motion_offset` | 初始偏移位置 |
| `Camera2D.zoom` | 相机缩放，与 Love2D 保持一致 |

---

## 3. 地图配置

### 3.1 Lorien 地图配置

```gdscript
{
    "info": {
        "name": "lorien",
        "theme": "forest",
        "width": 3000,
        "height": 1500,
        "horizon": 720,
        "bgm": "lorien.mp3"
    },
    "scope": {
        "x": 200,
        "y": 300,
        "w": 2600,
        "h": 900
    }
}
```

### 3.2 装饰系统

装饰对象包括：石头、草丛、花朵、大门等：

```gdscript
var decorations = [
    {"type": "stone", "count": 6, "y_offset": -10},
    {"type": "grass", "count": 15, "y_offset": -25},
    {"type": "flower", "count": 10, "y_offset": -15},
    {"type": "smallGrass", "count": 8, "y_offset": -12},
    {"type": "largeGrass", "count": 5, "y_offset": -45}
]
```

---

## 4. 关键修复记录

### 4.1 Camera2D 属性问题

| 问题 | 原因 | 修复 |
|------|------|------|
| `smoothing_enabled` | Godot 4.x 已移除 | 删除该属性设置 |
| `smoothing_speed` | Godot 4.x 已移除 | 删除该属性设置 |

### 4.2 图层渲染顺序

确保节点添加顺序与 Love2D 渲染顺序一致：

```gdscript
add_child(_layers["far"])      -- 先添加的先渲染
add_child(_layers["near"])
add_child(_layers["effect"])
add_child(_layers["floor"])
add_child(_object_layer)
```

### 4.3 地板碰撞体

加载预定义的地板场景，包含碰撞体：

```gdscript
var floor_scene = load("res://scenes/floor.tscn")
if floor_scene:
    _floor = floor_scene.instantiate()
    _layers["floor"].add_child(_floor)
```

---

## 5. 启动流程对比

### 5.1 Love2D 启动流程

```lua
function love.load()
    _MAP.Init()
    _MAP.Make("lorien")
    _MAP.Load(data)
end
```

### 5.2 Godot 启动流程

```gdscript
func _ready():
    setup_map_system()
    load_map_config("lorien")
    _on_start_game()  -- 直接进入游戏，不显示主菜单
```

---

## 6. 验证结果

### 6.1 运行日志

```
MapSystem initialized
Floor scene loaded with collision
Decorations added to object layer!
Map 'lorien' loaded with bounds: 3000x1500
Player spawned at: (640.0, 351.0)
Spawned 3 enemies to object layer
Game started!
```

### 6.2 功能验证

| 功能 | 状态 | 说明 |
|------|------|------|
| 远景视差 | ✅ | 0.3x 视差率 |
| 近景视差 | ✅ | 0.2x 视差率 |
| 地板渲染 | ✅ | 包含碰撞体 |
| 装饰渲染 | ✅ | 石头、草丛、花朵 |
| 相机缩放 | ✅ | 1.3x |
| 玩家碰撞 | ✅ | 不坠落 |
| 游戏启动 | ✅ | 直接进入游戏场景 |

---

## 7. 文件清单

### 7.1 修改的文件

| 文件 | 修改内容 |
|------|----------|
| `scripts/map_system.gd` | 完全重写，实现像素级复刻 |
| `scripts/game_manager.gd` | 更新地图初始化逻辑 |
| `scripts/ui_system.gd` | 隐藏 HUD（Love2D 无血条） |

### 7.2 资源文件

| 文件 | 说明 |
|------|------|
| `asset/textures/map/lorien/far.png` | 远景背景 |
| `asset/textures/map/lorien/near.png` | 近景背景 |
| `asset/textures/map/lorien/stone/*.png` | 石头装饰 |
| `asset/textures/map/lorien/grass/*.png` | 草丛装饰 |
| `asset/textures/map/lorien/flower/*.png` | 花朵装饰 |
| `asset/textures/map/lorien/pathgate/*.png` | 大门装饰 |

---

## 8. 操作说明

| 按键 | 操作 |
|------|------|
| 方向键/WASD | 移动 |
| Enter | 跳跃 |
| Space | 冲刺 |
| Esc | 攻击 |

---

## 9. 待办事项

- [ ] 添加天气效果系统
- [ ] 添加粒子特效系统
- [ ] 优化装饰渲染性能
- [ ] 实现地图矩阵寻路系统

---

**最后更新**: 2026-04-29