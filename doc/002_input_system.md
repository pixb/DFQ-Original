# 阶段 2: 输入系统

**目标**: 移植键盘、鼠标、触摸输入系统

---

## 任务清单

- [x] 2.1 InputManager (Autoload)
- [x] 2.2 键盘 (keyboard.lua) - 使用 Godot Input
- [x] 2.3 鼠标 (mouse.lua) - 使用 InputEvent
- [x] 2.4 触摸 (touch.lua) - Godot 支持
- [x] 2.5 配置 Input Map

---

## 2.1 InputManager

### LÖVE 结构

| 文件 | 功能 |
|------|------|
| `input.lua` | 状态机 (pressed/hold/released) |
| `keyboard.lua` | 键盘事件 + 监听器 |
| `mouse.lua` | 鼠标位置 + 按钮 |
| `touch.lua` | 多点触控 |

### Godot 等效

使用 Godot 内置 `Input` + 自定义 InputManager Autoload:

```gdscript
# scripts/input_manager.gd
extends Node

# Input state enum
enum InputState { PRESSED = 0, HOLD = 1, RELEASED = 2 }

# Key states
var _key_states: Dictionary = {}
var _mouse_pos: Vector2 = Vector2.ZERO
var _mouse_delta: Vector2 = Vector2.ZERO
var _mouse_pressed: int = -1
var _mouse_released: int = -1

# Touch points
var _touch_points: Dictionary = {}

# Callbacks
signal key_pressed(key: String)
signal key_released(key: String)
signal mouse_pressed(button: int, pos: Vector2)
signal mouse_released(button: int, pos: Vector2)
signal mouse_moved(pos: Vector2, delta: Vector2)
signal touch_pressed(id: int, pos: Vector2, delta: Vector2)
signal touch_released(id: int, pos: Vector2, delta: Vector2)
signal touch_moved(id: int, pos: Vector2, delta: Vector2)

func _ready() -> void:
	print("InputManager initialized")

func _input(event: InputEvent) -> void:
	match event.get_class():
		"InputEventKey":
			_handle_key_event(event)
		"InputEventMouseButton":
			_handle_mouse_button(event)
		"InputEventMouseMotion":
			_handle_mouse_motion(event)
		"InputEventScreenTouch":
			_handle_touch_event(event)
		"InputEventScreenDrag":
			_handle_touch_drag(event)

func _handle_key_event(event: InputEventKey) -> void:
	var key_name: String = event.key_label
	if event.pressed:
		_key_states[key_name] = InputState.PRESSED
		key_pressed.emit(key_name)
	else:
		_key_states[key_name] = InputState.RELEASED
		key_released.emit(key_name)

func _handle_mouse_button(event: InputEventMouseButton) -> void:
	var button_idx: int = event.button_index
	if event.pressed:
		_mouse_pressed = button_idx
		mouse_pressed.emit(button_idx, event.position)
	else:
		_mouse_released = button_idx
		mouse_released.emit(button_idx, event.position)

func _handle_mouse_motion(event: InputEventMouseMotion) -> void:
	_mouse_pos = event.position
	_mouse_delta = event.relative
	mouse_moved.emit(event.position, event.relative)

func _handle_touch_event(event: InputEventScreenTouch) -> void:
	var id: int = event.get_index()
	if event.pressed:
		_touch_points[id] = {
			"position": event.position,
			"delta": Vector2.ZERO,
			"status": InputState.PRESSED
		}
		touch_pressed.emit(id, event.position, Vector2.ZERO)
	else:
		_touch_points.erase(id)
		touch_released.emit(id, event.position, Vector2.ZERO)

func _handle_touch_drag(event: InputEventScreenDrag) -> void:
	var id: int = event.get_index()
	if _touch_points.has(id):
		_touch_points[id]["position"] = event.position
		_touch_points[id]["delta"] = event.relative
		touch_moved.emit(id, event.position, event.relative)

# Public API
func is_key_pressed(key: String) -> bool:
	return _key_states.get(key) == InputState.PRESSED

func is_key_held(key: String) -> bool:
	return _key_states.get(key) == InputState.HOLD

func is_key_released(key: String) -> bool:
	return _key_states.get(key) == InputState.RELEASED

func get_mouse_position(screen_scale: Vector2 = Vector2.ONE) -> Vector2:
	return _mouse_pos / screen_scale

func get_mouse_delta(screen_scale: Vector2 = Vector2.ONE) -> Vector2:
	return _mouse_delta / screen_scale

func get_mouse_button() -> int:
	return _mouse_pressed

func get_touch_points() -> Dictionary:
	return _touch_points

func is_action_pressed(action: StringName) -> bool:
	return Input.is_action_pressed(action)

func is_action_just_pressed(action: StringName) -> bool:
	return Input.is_action_just_pressed(action)

func is_action_just_released(action: StringName) -> bool:
	return Input.is_action_just_released(action)

func _process(delta: float) -> void:
	# Update key states
	var to_remove: Array[String] = []
	for key in _key_states:
		match _key_states[key]:
			InputState.PRESSED:
				_key_states[key] = InputState.HOLD
			InputState.RELEASED:
				to_remove.append(key)
	for key in to_remove:
		_key_states.erase(key)
	
	# Reset per-frame states
	_mouse_pressed = -1
	_mouse_released = -1
	_mouse_delta = Vector2.ZERO
	
	# Update touch states
	for id in _touch_points:
		var touch = _touch_points[id]
		match touch["status"]:
			InputState.PRESSED:
				touch["status"] = InputState.HOLD
			InputState.RELEASED:
				_touch_points.erase(id)
		touch["delta"] = Vector2.ZERO
```

---

## 2.2 键盘 (keyboard.lua)

| LÖVE | Godot |
|------|-------|
| `_KEYBOARD.IsPressed(key)` | `InputManager.is_key_pressed(key)` |
| `_KEYBOARD.IsHold(key)` | `InputManager.is_key_held(key)` |
| `_KEYBOARD.IsReleased(key)` | `InputManager.is_key_released(key)` |
| `key_pressed` callback | Signal |

### 使用

```gdscript
# 连接信号
InputManager.key_pressed.connect(_on_key_pressed)

func _on_key_pressed(key: String) -> void:
	print("Key pressed: ", key)
```

---

## 2.3 鼠标 (mouse.lua)

| LÖVE | Godot |
|------|-------|
| `_MOUSE.GetPosition()` | `InputManager.get_mouse_position()` |
| `_MOUSE.GetMoving()` | `InputManager.get_mouse_delta()` |
| `_MOUSE.IsPressed(btn)` | `Input.get_mouse_button()` |
| `onMoved` callback | Signal |

---

## 2.4 触摸 (touch.lua)

| LÖVE | Godot |
|------|-------|
| `_TOUCH.GetPoints()` | `InputManager.get_touch_points()` |
| `_TOUCH.GetPoint(id)` | `InputManager.get_touch_points()[id]` |
| `onPressed` callback | Signal |

---

## 2.5 Input Map 配置

在 Godot 中配置 `project.godot`:

```gdscript
[input]

ui_left={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194319,"key_label":0,"unicode":0,"echo":false,"script":null)
]
}
ui_right={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194321,"key_label":0,"unicode":0,"echo":false,"script":null)
]
}
ui_up={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194320,"key_label":0,"unicode":0,"echo":false,"script":null)
]
}
ui_down={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":4194322,"key_label":0,"unicode":0,"echo":false,"script":null)
]
}
move_left={
"deadzone": 0.5,
"events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"window_id":0,"alt_pressed":false,"shift_pressed":false,"ctrl_pressed":false,"meta_pressed":false,"pressed":false,"keycode":0,"physical_keycode":87,"key_label":0,"unicode":0,"echo":false,"script":null)
]
}
```

---

## 总结

- 替换 LÖVE 手动轮询为 Godot Signal
- 使用 Autoload 单例模式
- 保持原有 API 兼容

**预计工作量**: 2-3 小时

---

*阶段 2 完成*