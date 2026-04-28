# 阶段 10: Buff 系统

**目标**: 移植 buff 增益系统

---

## 任务清单

- [x] 10.1 Buff 基类
- [x] 10.2 Buff 组件
- [x] 10.3 BuffService
- [x] 10.4 具体 Buff (Speed)

---

## 已创建

```
scripts/
├── buff_base.gd          # Buff 基类
├── buff_component.gd     # Buff 组件
├── buff_service.gd      # Buff 服务
└── buff_speed.gd       # Speed Buff
```

## BuffBase

```gdscript
class_name BuffBase
extends RefCounted

var entity: Node
var data: Dictionary
var is_running: bool = true
var has_exited: bool = false
var _remaining_time: float = 0.0

func on_update(delta)
func on_late_update(delta)
func update(delta)
func late_update(delta)
func exit() -> bool
func get_data()
func get_path() -> String
func is_only() -> bool
func is_debuff() -> bool
func get_process() -> float
func set_time(time)
```

## BuffComponent

```gdscript
class_name BuffComponent
extends RefCounted

var entity: Node
var list: Array = []
var enabled: bool = true
var undebuff_count: int = 0

func add_buff(buff)
func remove_buff(buff)
func get_buff(path, tag)
func get_buff_count(path) -> int
func has_buff(path) -> bool
func update(delta)
func late_update(delta)
func clear_debuff()
func clear_all()
```

## BuffService

创建和管理 Buff:

```gdscript
class_name BuffService
extends RefCounted

static func add_buff(entity, data) -> BuffBase
static func get_buff(buffs, path, tag)
static func get_buff_count(buffs, path) -> int
static func clear_debuff(buffs)
static func clear_all(buffs)
```

## 测试结果

```
Buff added: true
Buff count: 1
=== Phase 10 Complete ===
```

---

*阶段 10 完成*