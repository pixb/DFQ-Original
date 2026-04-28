# Phase 27: 敌人精灵图实现

## 概述

本阶段实现敌人（Goblin）精灵图显示，替换调试矩形为真实精灵纹理。

## 完成任务

### 1. 敌人精灵图实现
- **状态**: ✅ 已完成
- **文件**: `asset/textures/goblin.png`

### 2. 场景修改
- **状态**: ✅ 已完成
- **修改文件**: `scenes/enemy.tscn`
- **变更**: 替换 ColorRect 为 Sprite2D

### 3. 脚本修改
- **状态**: ✅ 已完成
- **修改文件**: `scripts/enemy.gd`
- **变更**: 添加 sprite 初始化和翻转控制

### 4. 技术实现

#### 场景修改 (enemy.tscn)

```tscn
[gd_scene format=3 uid="uid://tq2nkhj3ad86"]

[ext_resource type="Script" uid="uid://cjciqa2jj1cv7" path="res://scripts/enemy.gd" id="1_7p1mj"]
[ext_resource type="Texture2D" path="res://asset/textures/goblin.png" id="2_goblin"]

[sub_resource type="CapsuleShape2D" id="CapsuleShape2D_enemy"]
radius = 16.0
height = 48.0

[node name="root" type="CharacterBody2D" unique_name_uid=true]
script = ExtResource("1_7p1mj")

[node name="CollisionShape2D" type="CollisionShape2D" parent="."]
shape = SubResource("CapsuleShape2D_enemy")

[node name="Sprite2D" type="Sprite2D" parent="."]
texture = ExtResource("2_goblin")
flip_h = false
```

**注意**: 在 tscn 文件中不能直接使用 `load()` 函数，必须通过 `ext_resource` 定义外部资源，然后用 `ExtResource()` 引用。

#### 脚本修改 (enemy.gd)

```gdscript
var sprite: Sprite2D = null

func _ready():
    start_position = global_position
    current_state = EnemyState.IDLE
    sprite = $Sprite2D
    if sprite:
        print("Enemy sprite initialized")

func _physics_process(delta: float) -> void:
    update_ai(delta)
    update_sprite()
    move_and_slide()

func update_sprite() -> void:
    if not sprite:
        return
    
    if velocity.x != 0:
        sprite.flip_h = velocity.x < 0
```

### 5. 技术说明

敌人精灵系统比玩家更复杂，原项目使用多层精灵系统：

| 图层 | 说明 |
|------|------|
| skin | 皮肤层 |
| pants | 裤子层 |
| weapon | 武器层 |
| shoulder | 肩部装备 |
| coat | 外套层 |
| hair | 头发层 |

当前实现仅使用 skin/base/0.png 作为基础纹理，完整的多层精灵系统可以在后续阶段实现。

### 6. 功能特性

| 特性 | 说明 |
|------|------|
| 纹理加载 | 使用 goblin.png 作为精灵纹理 |
| 水平翻转 | 根据移动方向自动翻转精灵 |
| 基础显示 | 单图层显示（完整多层系统待实现） |

## 相关文件

- `asset/textures/goblin.png` - 敌人精灵纹理
- `scenes/enemy.tscn` - 敌人场景
- `scripts/enemy.gd` - 敌人控制器

---

*创建日期: 2026-04-28*