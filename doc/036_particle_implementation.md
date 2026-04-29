# Phase 34: 粒子效果系统实现

## 概述

本阶段实现Godot粒子效果系统，使用LÖVE2D原项目的特效资源。

## 资源目录

原项目特效资源位于：`/Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/`

### 特效类型

| 类别 | 子目录 | 描述 |
|------|--------|------|
| 攻击特效 | hitting/fire | 火焰攻击特效 |
| 攻击特效 | hitting/dark | 暗黑攻击特效 |
| 攻击特效 | hitting/water | 水波攻击特效 |
| 攻击特效 | hitting/light | 圣光攻击特效 |
| 攻击特效 | hitting/smallSlash | 小型斩击 |
| 攻击特效 | hitting/largeSlash | 大型斩击 |
| Buff特效 | buff/bleed | 流血效果 |
| Buff特效 | buff/fear | 恐惧效果 |
| Buff特效 | buff/freeze | 冰冻效果 |
| Buff特效 | buff/haste | 加速效果 |
| 死亡特效 | death/normal | 普通死亡 |
| 召唤特效 | summon/back | 背景召唤 |
| 召唤特效 | summon/front | 前方召唤 |
| 召唤特效 | summon/bottom | 底部召唤 |

## 实现方案

### 1. 特效管理器

```gdscript
# scripts/effect_manager.gd
class_name EffectManager
extends Node

var effect_pool: Dictionary = {}

func _ready():
    preload_effects()

func preload_effects():
    # 预加载常用特效
    var hit_effects = ["fire", "dark", "water", "light"]
    for effect_name in hit_effects:
        var scene = load("res://scenes/effects/hit_" + effect_name + ".tscn")
        if scene:
            effect_pool[effect_name] = scene
            print("Preloaded effect: ", effect_name)

func play_hit_effect(effect_type: String, position: Vector2):
    if effect_pool.has(effect_type):
        var effect = effect_pool[effect_type].instantiate()
        effect.position = position
        get_tree().current_scene.add_child(effect)

func play_buff_effect(buff_type: String, target: Node2D):
    var buff_scene = load("res://scenes/effects/buff_" + buff_type + ".tscn")
    if buff_scene:
        var buff = buff_scene.instantiate()
        buff.position = target.position
        buff.follow_target = target
        get_tree().current_scene.add_child(buff)

func play_death_effect(position: Vector2):
    var death_scene = load("res://scenes/effects/death_normal.tscn")
    if death_scene:
        var death = death_scene.instantiate()
        death.position = position
        get_tree().current_scene.add_child(death)
```

### 2. 特效场景创建

#### 攻击特效场景模板

```gd_scene
[gd_scene format=3]

[node name="HitEffect" type="Node2D"]

[node name="Sprite2D" type="AnimatedSprite2D" parent="."]
frames = SubResource("SpriteFrames_effect")
animation = "default"
z_index = 10

[sub_resource type="SpriteFrames" id="SpriteFrames_effect"]
animations = [
{
  "name": "default",
  "speed": 15.0,
  "loop": false,
  "frames": [
    ExtResource("1_frame0"),
    ExtResource("1_frame1"),
    ExtResource("1_frame2"),
    ExtResource("1_frame3"),
    ExtResource("1_frame4")
  ]
}
]

[ext_resource type="Texture2D" path="res://asset/effects/hitting/fire/0.png" id="1_frame0"]
[ext_resource type="Texture2D" path="res://asset/effects/hitting/fire/1.png" id="1_frame1"]
[ext_resource type="Texture2D" path="res://asset/effects/hitting/fire/2.png" id="1_frame2"]
[ext_resource type="Texture2D" path="res://asset/effects/hitting/fire/3.png" id="1_frame3"]
[ext_resource type="Texture2D" path="res://asset/effects/hitting/fire/4.png" id="1_frame4"]
```

### 3. 特效基础类

```gdscript
# scripts/effect_base.gd
class_name EffectBase
extends Node2D

signal effect_finished

var sprite: AnimatedSprite2D
var animation_player: AnimationPlayer
var follow_target: Node2D = null

func _ready():
    sprite = $Sprite2D
    if sprite:
        sprite.animation_finished.connect(_on_animation_finished)

func _process(delta):
    if follow_target and is_instance_valid(follow_target):
        position = follow_target.position

func _on_animation_finished():
    effect_finished.emit()
    queue_free()
```

## 集成到游戏管理器

```gdscript
# game_manager.gd
var effect_manager: EffectManager

func _ready():
    # ... 其他初始化 ...
    setup_effect_manager()

func setup_effect_manager():
    effect_manager = EffectManager.new()
    effect_manager.name = "EffectManager"
    add_child(effect_manager)

func on_player_attack():
    var hit_position = player.position + Vector2(50, 0)
    effect_manager.play_hit_effect("fire", hit_position)

func on_enemy_damaged(enemy, damage):
    effect_manager.play_hit_effect("dark", enemy.position)
    show_damage_number(damage, enemy.position)

func on_player_death():
    effect_manager.play_death_effect(player.position)
```

## 实现步骤

### 步骤 1: 创建特效目录

```bash
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/scenes/effects
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/hitting
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/buff
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/death
```

### 步骤 2: 复制特效资源

```bash
# 攻击特效
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/hitting/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/hitting/

# Buff特效
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/buff/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/buff/

# 死亡特效
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/death/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/death/
```

### 步骤 3: 创建特效场景

在 Godot 编辑器中创建以下场景：
- `scenes/effects/hit_fire.tscn`
- `scenes/effects/hit_dark.tscn`
- `scenes/effects/hit_water.tscn`
- `scenes/effects/hit_light.tscn`
- `scenes/effects/buff_bleed.tscn`
- `scenes/effects/buff_freeze.tscn`
- `scenes/effects/death_normal.tscn`

### 步骤 4: 创建特效脚本

```bash
cp /Volumes/data/dev/code/game/DFQ-Original/doc/effect_manager_gd.md /Volumes/data/dev/code/game/DFQ-Original/dfq/scripts/effect_manager.gd
cp /Volumes/data/dev/code/game/DFQ-Original/doc/effect_base_gd.md /Volumes/data/dev/code/game/DFQ-Original/dfq/scripts/effect_base.gd
```

## 测试验证

### 测试用例

| 测试项 | 操作 | 预期结果 |
|--------|------|----------|
| 攻击特效 | 按下攻击键 | 显示攻击特效 |
| 受伤特效 | 敌人受到攻击 | 显示受伤特效 |
| 死亡特效 | 角色死亡 | 显示死亡特效 |
| Buff特效 | 应用Buff | 特效跟随角色 |

### 调试输出

```gdscript
# 在effect_manager.gd中添加调试
func play_hit_effect(effect_type, position):
    print("Playing effect: ", effect_type, " at ", position)
```

## 性能优化

### 对象池优化

```gdscript
# 在effect_manager.gd中实现对象池
var effect_pool: Dictionary = {}

func get_effect(effect_type: String):
    if effect_pool.has(effect_type) and effect_pool[effect_type].size() > 0:
        return effect_pool[effect_type].pop_back()
    
    var scene = load("res://scenes/effects/" + effect_type + ".tscn")
    return scene.instantiate()

func return_effect(effect_type: String, effect: Node2D):
    if not effect_pool.has(effect_type):
        effect_pool[effect_type] = []
    effect_pool[effect_type].push_back(effect)
    effect.visible = false
```

### 批量渲染

使用 `CanvasItem` 的 `draw` 方法批量渲染粒子。

## 后续改进

- ✅ Phase 35: 高级粒子效果
- ✅ Phase 36: 天气特效系统
- ✅ Phase 37: 特效性能优化

## 相关文档

- effect_manager_gd.md - 特效管理器代码模板
- effect_base_gd.md - 特效基础类代码模板
- 034_particle_effects.md - 粒子效果系统规划
