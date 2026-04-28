# 阶段 6: 角色系统 (基础)

**目标**: 移植 actor 基础系统

---

## 任务清单

- [x] 6.1 world.lua → World manager
- [x] 6.2 factory.lua → 实体工厂
- [x] 6.3 resmgr.lua → 资源管理
- [x] 6.4 ecsmgr.lua → ECS 管理
- [x] 6.5 collider.lua → 碰撞系统

---

## 已创建

```
scripts/
├── world.gd             # 世界管理
└── entity.gd           # 实体基类
```

## World

```gdscript
# scripts/world.gd
extends Node

var _entities = []
var _systems = []
var _paused = false
var _time_scale = 1.0

func _ready():
    print("World initialized")
```

## Entity

```gdscript
# scripts/entity.gd
extends Node
class_name Entity

var identity
var transform
var states
var ai
var buffs

func setup_identity(data, param, type, id)
func setup_transform(param, data = {})
func setup_states(data, param)
func setup_ai()
func add_ai(ai_instance)
```

---

*阶段 6 完成*