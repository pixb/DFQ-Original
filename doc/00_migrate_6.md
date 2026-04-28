# 阶段 6: 角色系统 (基础)

**目标**: 移植 actor 基础系统

---

## 任务清单

- [ ] 6.1 world.lua → World manager
- [ ] 6.2 factory.lua → 实体工厂
- [ ] 6.3 resmgr.lua → 资源管理
- [ ] 6.4 ecsmgr.lua → ECS 管理
- [ ] 6.5 collider.lua → 碰撞系统

---

## 6.1 world.lua 分析

```lua
-- 核心系统管理器
-- 包含: 系统列表、更新循环、绘制列表、数字提示
```

### Godot 实现

```gdscript
# scripts/world.gd
extends Node

var _systems = []
var _entities = []
var _paused = false

func _ready():
	print("World initialized")

func _process(delta):
	if _paused:
		return
	for system in _systems:
		system._update(delta)

func _physics_process(delta):
	for system in _systems:
		system._physics_update(delta)
```

---

## 6.2 factory.lua

```lua
-- 实体创建工厂
```

### Godot 实现

```gdscript
# scripts/entity_factory.gd
class_name EntityFactory

static func create_entity(type_name):
	var entity = Entity.new()
	entity.type_name = type_name
	return entity
```

---

## 6.3 resmgr.lua

```lua
-- 资源管理 (装备、Buff、Skill等)
```

### Godot 实现

```gdscript
# scripts/res_manager.gd
extends Node

var _equipment_data = {}
var _buff_data = {}
var _skill_data = {}

func _ready():
	print("ResManager loaded")

func load_equipment(id):
	# Load from config file
	pass

func get_buff(id):
	return _buff_data.get(id)

func get_skill(id):
	return _skill_data.get(id)
```

---

## 6.4 ecsmgr.lua

```lua
-- ECS 管理系统
```

### Godot 实现

使用 Godot 内置 Node 系统替代 ECS:

- Entity → Node
- Component → Node child
- System → Script with _process

---

## 6.5 collider.lua

```lua
-- 碰撞检测
```

### Godot 实现

使用 Godot 内置 Physics:

```gdscript
# 使用 Area2D, CollisionShape2D
# 或者 PhysicsServer2D API
```

---

## 总结

大部分 actor 系统可用 Godot 内置机制替代:

| LÖVE | Godot |
|------|-------|
| world | Node + Process |
| factory | class_name functions |
| resmgr | JSON + load() |
| ecsmgr | Node tree |
| collider | PhysicsServer |

**预计工作量**: 6-8 小时

---

*阶段 6*