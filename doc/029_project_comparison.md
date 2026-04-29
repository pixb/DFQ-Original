# LÖVE vs Godot 项目逻辑比较分析

## 1. 入口点与游戏循环

### LÖVE 原项目
```lua
function love.load()
    _GRAPHICS.Init()
    _DIRECTOR.Init()
end

function love.update(dt)
    _TIME.Update(dt)
    while (_TIME.CanUpdate()) do
        _Update()
    end
end
```

### Godot 项目
```gdscript
// project.godot autoload
[autoload]
InputManager="*res://scripts/input_manager.gd"
AudioManager="*res://scripts/audio_manager.gd"

// scenes/game.tscn
func _ready():
    setup_camera()
    setup_floor()
    setup_player()
```

---

## 2. 核心系统架构

### LÖVE - Director 模式
```lua
_DIRECTOR = {rate = 1}
function _DIRECTOR.Update(dt)
    _WORLD.Update(dt, _DIRECTOR.rate)
    _MAP.Update(dt)
end
```

### Godot - GameManager 模式
```gdscript
class_name GameManager extends Node2D
enum GameState { PLAYING, PAUSED, GAME_OVER, VICTORY }
```

---

## 3. 组件与实体系统

### LÖVE - 自定义ECS
```lua
_Entity = require("core.class")()
function _Entity:Ctor()
    self.identity = _Identity.New()
    self.transform = _Transform.New()
end
```

### Godot - Node组件模式
```gdscript
class_name Entity extends Node
var identity
var transform
func get_component(comp_name: String):
    return _component_map.get(comp_name)
```

---

## 4. 输入系统

### LÖVE
```lua
local input = require("lib.input")
if input.IsPressed("move_left") then end
```

### Godot
```gdscript
if Input.is_action_pressed("move_left"):
    velocity.x = -SPEED
```

---

## 5. 音频系统

### LÖVE
```lua
_MUSIC.Play(path, fade_time, loop)
```

### Godot
```gdscript
audio_manager.play_music("res://asset/music/lorien.mp3", 1.0, true)
```

---

## 6. UI系统

### LÖVE - Layer绘制
```lua
_Layer:Add(tag, order, Func, ...)
```

### Godot - Control节点
```gdscript
[node name="VBoxContainer" type="VBoxContainer" parent="."]
[node name="StartButton" type="Button" parent="VBoxContainer"]
```

---

## 总结对比表

| 功能 | LÖVE 原项目 | Godot 项目 | 迁移状态 |
|------|------------|------------|----------|
| 入口点 | love.load() | _ready() | ✅ 完成 |
| 游戏循环 | 手动while循环 | _process() | ✅ 完成 |
| 核心管理 | Director单例 | GameManager | ✅ 完成 |
| 实体系统 | 自定义ECS | Entity Node | ✅ 完成 |
| 组件系统 | 类继承组件 | Node Component | ✅ 完成 |
| 输入系统 | lib.input | InputMap | ✅ 完成 |
| 音频系统 | lib.music | AudioStreamPlayer | ✅ 完成 |
| UI系统 | Layer绘制 | Control节点 | ✅ 完成 |
| 地图系统 | Matrix动态加载 | TileMap/场景 | ⚠️ 简化版 |
| 碰撞系统 | 自定义Matrix | Godot Physics | ⚠️ 简化版 |
| 动画系统 | Frameani帧动画 | AnimatedSprite2D | ⚠️ 简化版 |
| 技能系统 | 完整技能组件 | 框架存在 | ⚠️ 未集成 |
| Buff系统 | 完整Buff组件 | 框架存在 | ⚠️ 未集成 |
| AI系统 | 状态机AI | 简化版 | ⚠️ 简化版 |

---

## 剩余迁移工作

### 高优先级
1. **完整碰撞系统** - TileMap碰撞
2. **动画系统完善** - 多帧动画支持
3. **技能系统集成** - 与玩家控制器集成

### 中优先级
4. **地图编辑器支持** - TileMap配置
5. **完整AI状态机** - 敌人行为完善
6. **Buff/DEBUFF系统** - 状态效果集成

### 低优先级
7. **粒子系统** - 特效渲染
8. **Shader效果** - 画面滤镜
9. **存档系统** - 配置读写
