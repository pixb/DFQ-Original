# 视差背景系统

## 概述

实现了LÖVE原项目的多层视差背景效果，使用CanvasLayer和ColorRect创建基本的森林场景。

## LÖVE原项目分析

### 核心文件

**source/map/background.lua**
```lua
---@class MAP.Background:Graphics.Drawable.Sprite
---@field public rate number
local _Background = require("core.class")(_Sprite)

function _Background:_OnDraw()
    _GRAPHICS.Push()
    _GRAPHICS.Translate(-self._upperEvent.GetShift() * self.rate, 0)
    _Sprite._OnDraw(self)
    _GRAPHICS.Pop()
end
```

**source/actor/drawable/layer.lua**
```lua
---@class Graphics.Drawable.Layer:Graphics.Drawable
local _Layer = require("core.class")(_Base, _Container)

function _Layer:Update(dt)
    for n=1, #self._list do
        self._list[n]:Update(dt)
    end
end
```

### 视差原理

使用不同的`rate`值实现多层视差：
- 背景层（天空）: rate = 0（固定）
- 中层（山脉）: rate = 0.3（慢速）
- 前景（森林）: rate = 0.7（快速）

## Godot实现

### 文件结构

| 文件 | 功能 |
|------|------|
| `scripts/background_layer.gd` | 视差层基类 |
| `scenes/parallax_background.tscn` | 多层背景场景 |
| `scripts/game_manager.gd` | 集成背景系统 |

### background_layer.gd

```gdscript
class_name BackgroundLayer
extends Node2D

var sprite_path: String = ""
var texture: Texture2D = null
var parallax: float = 0.3
var offset: Vector2 = Vector2.ZERO
var scale: float = 1.0
var is_repeat: bool = false

func _draw() -> void:
    if texture:
        var draw_pos = offset
        if is_repeat:
            draw_texture_rect(texture, Rect2(draw_pos, Vector2(2560, 720) * scale), true)
        else:
            draw_texture(texture, draw_pos)
```

### parallax_background.tscn

场景结构：
```
ParallaxBackground (CanvasLayer - layer -1)
├── SkyLayer (ColorRect - 天蓝色 - 全屏)
├── MountainLayer (ColorRect - 灰蓝色 - Y=250)
├── ForestBackLayer (ColorRect - 深绿色 - Y=350)
└── ForestFrontLayer (ColorRect - 暗绿色 - Y=450)
```

颜色配置：
```
天空: Color(0.529, 0.808, 0.922)
山脉: Color(0.35, 0.35, 0.45)
后林: Color(0.15, 0.4, 0.15)
前林: Color(0.08, 0.25, 0.08)
```

### game_manager.gd集成

```gdscript
func _ready():
    print("Game started!")
    setup_camera()
    setup_background()        # ← 新增
    setup_floor()
    setup_player()
    setup_enemies()
    setup_health_bar()
    setup_damage_number_system()
    setup_game_over_screen()
    play_background_music()

func setup_background():
    var bg_scene = load("res://scenes/parallax_background.tscn")
    var bg = bg_scene.instantiate()
    add_child(bg)
    print("Parallax background loaded")
```

## 测试结果

```
✅ InputHandler initialized
✅ AudioManager initialized
✅ Game started!
✅ Parallax background loaded ← 新增
✅ Floor created at: (640.0, 500.0)
✅ Player sprite initialized
✅ Player spawned at: (640.0, 360.0)
✅ Enemy sprite initialized (x3)
✅ Spawned 3 enemies
✅ Damage number system ready
✅ Game over screen ready
✅ Background music started: lorien.mp3
```

## 后续改进方向

### Phase 31: 完整视差效果
- 根据相机移动计算各层偏移
- 实现真正的视差滚动
- 添加纹理替代纯色

### Phase 32: 场景装饰元素
- 添加拱门（像原项目截图）
- 添加树木、石头等装饰物
- 实现装饰层管理

### Phase 33: 完整地图系统
- 使用TileMap替代简化地板
- 实现Matrix碰撞检测
- 添加地图加载逻辑

## 迁移状态

- ✅ 基本背景系统实现
- ✅ 多层场景结构
- ⚠️ 视差效果未完全实现（无相机跟踪）
- ⚠️ 使用纯色而非实际纹理

---
*Last Updated: 2026-04-29*
