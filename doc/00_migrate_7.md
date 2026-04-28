# 阶段 7: Component 系统

**目标**: 移植 component 系统

---

## 任务清单

- [x] 7.1 Component 基类
- [x] 7.2 Identity 组件
- [x] 7.3 Transform 组件
- [x] 7.4 States 组件
- [x] 7.5 其他组件

---

## 已创建

```
scripts/
├── component.gd              # 组件基类
├── identity_component.gd    # Identity 组件
├── transform_component.gd   # Transform 组件
├── state_component.gd    # States 组件
└── state.gd             # State 类
```

## Component 基类

```gdscript
class_name Component
extends RefCounted

var owner_node: Node
var enabled: bool = true

func _init(owner: Node):
    owner_node = owner

func on_enter()
func on_exit()
func on_update(delta: float)
```

## Identity 组件

存储实体标识数据:

```gdscript
class_name IdentityComponent
extends RefCounted

func setup(entity, data, param, id_type, id_val)
func get_path() -> String
func get_name() -> String
```

## Transform 组件

管理位置、方向、缩放:

```gdscript
class_name TransformComponent
extends RefCounted

var position: Vector3
var direction: int = 1
var scale: Vector2 = Vector2(1, 1)
var rotation: float = 0.0

func get_position_2d() -> Vector2
func set_position_2d(pos: Vector2)
```

## States 组件

管理状态机:

```gdscript
class_name StateComponent
extends RefCounted

var current
var first_state: String = "stay"
var map: Dictionary = {}

func set_state(state_name: String, force: bool = false)
func get_state() -> String
func has_state(state_name: String) -> bool
```

---

*阶段 7 完成*