# 阶段 9: AI 系统

**目标**: 移植 AI 决策系统

---

## 任务清单

- [x] 9.1 AI 基类
- [x] 9.2 AI 组件
- [x] 9.3 Judge AI

---

## 已创建

```
scripts/
├── ai_base.gd         # AI 基类
├── ai_component.gd    # AI 组件
└── ai_judge.gd       # Judge AI
```

## AiBase

```gdscript
class_name AiBase
extends RefCounted

var entity: Node
var enabled: bool = true
var active: bool = false

func can_run() -> bool
func tick() -> bool
func update(delta: float)
```

## AiComponent

```gdscript
class_name AiComponent
extends RefCounted

var entity: Node
var enabled: bool = true
var ai_list: Array = []
var active_ai

func add_ai(ai)
func remove_ai(ai)
func tick() -> bool
func update(delta: float)
func set_enabled(is_enabled)
```

## AiJudge

```gdscript
class_name AiJudge
extends AiBase

var collider
var action_key: String
var selector: Callable

func tick() -> bool
```

---

*阶段 9 完成*