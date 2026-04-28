# 阶段 5: 核心系统

**目标**: 移植 core 核心类系统

---

## 任务清单

- [x] 5.1 class.lua → Godot class
- [x] 5.2 caller.lua → Signal
- [x] 5.3 container.lua → Array/Dict
- [x] 5.4 quickList.lua → Array
- [x] 5.5 watchValue.lua → Property
- [x] 5.6 其他工具

---

## 5.1 class.lua

### LÖVE 类系统

```lua
local MyClass = require("core.class")()
function MyClass:Ctor()
    self.value = 0
end
function MyClass:GetValue()
    return self.value
end
return MyClass
```

### Godot 等效

直接使用 GDScript 继承:

```gdscript
# my_class.gd
class_name MyClass
extends RefCounted

var value: int = 0

func GetValue():
    return value
```

无需额外 class.lua, Godot 内置 `extends` 和 `class_name`。

---

## 5.2 caller.lua (Observer)

### LÖVE

```lua
local caller = require("core.caller").New()
caller:AddListener(obj, func)
caller:Call(arg1, arg2)
```

### Godot 等效

使用内置 Signal:

```gdscript
signal my_signal

func _ready():
    my_signal.connect(_on_signal)

func _on_signal(arg):
    print(arg)

func emit_signal():
    my_signal.emit("value")
```

| LÖVE | Godot |
|------|-------|
| Caller:AddListener() | signal.connect() |
| Caller:Call() | signal.emit() |
| Caller:DelListener() | signal.disconnect() |

---

## 5.3 container.lua

### LÖVE

```lua
local container = require("core.container").New()
container:Add(tag, object)
container:Get(tag)
container:Remove(tag)
```

### Godot 等效

使用 Array + Dictionary:

```gdscript
var _list = []
var _map = {}

func add(tag, object):
    _list.append(object)
    _map[tag] = object

func get(tag):
    return _map.get(tag)

func remove(tag):
    var obj = _map.get(tag)
    if obj:
        _list.erase(obj)
        _map.erase(tag)
    return obj
```

---

## 5.4 quickList.lua

### LÖVE

快速数组操作

### Godot 等效

直接使用 Array:

```gdscript
var arr = []
arr.append(item)
arr.has(item)
arr.insert(index, item)
arr.erase(item)
```

---

## 5.5 watchValue.lua

### LÖVE

观察值变化

### Godot 等效

使用 setter/getter:

```gdscript
var _health = 100

func set_health(value):
    if _health != value:
        var old = _health
        _health = value
        health_changed.emit(old, value)
    return _health

func get_health():
    return _health

signal health_changed(old, new)
```

---

## 5.6 其他工具

| 文件 | Godot 等效 |
|------|----------|
| activeObj.lua | Object pooling |
| gear.lua | Tween |
| quickList.lua | Array |
| watchValue.lua | Property + Signal |

---

## 总结

大部分核心系统可直接使用 Godot 内置机制:

- **类** → `extends` + `class_name`
- **观察者** → Signal
- **容器** → Array + Dictionary
- **工具** → GDScript 内置

**预计工作量**: 3-4 小时

---

*阶段 5 完成*