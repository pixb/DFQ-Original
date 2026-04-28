# 阶段 1: 基础项目搭建

**目标**: 建立 Godot 项目基础框架，实现最小可运行版本

---

## 任务清单

- [x] 1.1 创建 Godot 项目
- [x] 1.2 移植配置系统
- [x] 1.3 移植基础 lib 库
- [x] 1.4 创建主场景
- [x] 1.5 验证运行

---

## 1.1 创建 Godot 项目

### 操作

使用 Godot MCP 创建项目:

```bash
# 在项目根目录创建 Godot 项目
```

### 项目结构

```
DFQ-Original/
├── project.godot           # 项目配置
├── main.tscn              # 主场景
├── main.gd               # 主脚本
├── addons/               # 插件目录
├── scenes/              # 场景目录
├── scripts/             # 脚本目录
├── resources/          # 资源目录
├── assets/             # 保留原资源目录
└── source/             # 保留原代码 (参考)
```

### 配置 (project.godot)

```gdscript
config/version="0.1.0"
config/name="Dungeon Fighter Quest"
run/main_scene="res://main.tscn"
display/window/size/viewport_width=1280
display/window/size/viewport_height=720
display/window/stretch/mode="canvas_items"
display/window/stretch/aspect="expand"
display/window/high_dpif=true
```

---

## 1.2 移植配置系统

### LÖVE conf.lua

```lua
function love.conf (t)
    io.stdout:setvbuf("no")
    t.window.title = "Dungeon Fighter Quest"
    t.window.width = 1280
    t.window.height = 720
    t.window.highdpi = true
    t.window.borderless = true
end
```

### Godot 等效配置

| LÖVE | Godot |
|------|-------|
| window.title | application/config/name |
| window.width | display/window/size/viewport_width |
| window.height | display/window/size/viewport_height |
| window.highdpi | display/window/high_dpif |
| window.borderless | display/window/borderless |
| io.stdout:setvbuf | GDScript print() |

### 迁移步骤

1. 在 `project.godot` 中设置窗口尺寸
2. 创建 `project.gd` 脚本处理运行时配置

---

## 1.3 移植基础 lib 库

### 优先级

1. **time.lua** ⭐ 最重要 - 时间管理
2. **math.lua** 🔧 工具 - 数学工具
3. **string.lua** 🔧 工具 - 字符串工具
4. **table.lua** 🔧 工具 - 表格工具
5. **system.lua** 🔧 工具 - 系统工具

### 1.3.1 lib/time.lua

| 原函数 | Godot 等效 |
|--------|-----------|
| GetDelta() | Engine.get_process_delta_time() |
| GetElapsed() | Time.get_ticks_msec() / 1000.0 |
| CanUpdate() | 自定义 (帧率控制) |

### 迁移代码

```gdscript
# scripts/time.gd
extends RefCounted

var _delta: float = 0.0
var _elapsed: float = 0.0
var _fps: int = 60
var _accumulator: float = 0.0
var _fixed_delta: float = 1.0 / 60.0

func _init() -> void:
    _fixed_delta = 1.0 / _fps

func GetDelta() -> float:
    return _delta

func GetElapsed() -> float:
    return _elapsed

func CanUpdate() -> bool:
    return true

func Update(dt: float) -> void:
    _delta = dt
    _elapsed += dt

func FrameUpdate() -> void:
    _accumulator += _delta
    while _accumulator >= _fixed_delta:
        _accumulator -= _fixed_delta

func LateUpdate() -> void:
    pass
```

### 1.3.2 lib/math.lua

迁移为 `scripts/math_util.gd`:

| 原函数 | GDScript |
|--------|----------|
| Round() | round() |
| Floor() | floor() |
| Ceil() | ceil() |
| Clamp() | clamp() |
| Lerp() | lerp() |
| Rand() | randf() |
| RandRange() | randf_range() |

### 1.3.3 lib/string.lua

迁移为 `scripts/string_util.gd`:

| 原函数 | GDScript |
|--------|----------|
| Split() | split() |
| Trim() | strip_edges() |
| Format() | "%s" % value |

### 1.3.4 lib/table.lua

迁移为 `scripts/table_util.gd`:

| 原函数 | GDScript |
|--------|----------|
| Clone() | duplicate() |
| Clear() | clear() |
| Insert() | append() |
| Remove() | erase() |

---

## 1.4 创建主场景

### main.tscn

```xml
[gd_scene load_steps=2 format=3]

[ext_resource type="Script" path="res://scripts/main.gd" id="1"]

[node name="Main" type="Node2D"]
script = ExtResource("1")
```

### main.gd

```gdscript
extends Node2D

var _TIME = preload("res://scripts/time.gd").new()

func _ready() -> void:
    print("DFQ Initializing...")
    _TIME.Update(0.0)

func _process(delta: float) -> void:
    _TIME.Update(delta)
    
    while _TIME._accumulator >= _TIME._fixed_delta:
        _process_fixed(_TIME._fixed_delta)
        _TIME._accumulator -= _TIME._fixed_delta
    
    _draw()

func _process_fixed(delta: float) -> void:
    pass

func _draw() -> void:
    queue_redraw()
```

---

## 1.5 验证运行

### MCP 命令

```bash
# 运行项目
godot_run_project

# 获取调试输出
godot_get_debug_output
```

### 预期输出

```
DFQ Initializing...
```

---

## 总结

- 最小可运行版本
- Godot 项目基础结构
- 基础库框架
- 验证构建系统正常

**预计工作量**: 2-4 小时

---

*阶段 1 完成*