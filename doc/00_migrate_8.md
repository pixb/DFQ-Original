# 阶段 8: Service 系统

**目标**: 移植 service 服务系统

---

## 任务清单

- [x] 8.1 InputService
- [x] 8.2 MotionService
- [x] 8.3 StateService
- [x] 8.4 Services 管理器

---

## 已创建

```
scripts/
├── input_service.gd    # Input 服务
├── motion_service.gd   # Motion 服务
├── state_service.gd    # State 服务
└── services.gd       # Service 管理器
```

## InputService

```gdscript
class_name InputService
extends RefCounted

func is_pressed(action) -> bool
func is_held(action) -> bool
func is_released(action) -> bool
func get_axis(negative, positive) -> float
func get_direction(direction) -> Dictionary
```

## MotionService

```gdscript
class_name MotionService
extends RefCounted

const TILE_SIZE = 32.0

func move(transform, direction, distance)
func move_to(transform, target, speed, delta) -> bool
func shake(transform, intensity)
func aim_direction(a, b)
func turn_direction(transform, input_service, current_dir) -> bool
func adjust_collider(transform, collider, offset, scale)
```

## StateService

```gdscript
class_name StateService
extends RefCounted

func play(states, state_name, is_only = false) -> bool
func has_tag(states, key) -> bool
func reset(states, force = false) -> bool
func auto_play_end(states, aspect, next_state)
func reload_state_data(states)
```

---

*阶段 8 完成*