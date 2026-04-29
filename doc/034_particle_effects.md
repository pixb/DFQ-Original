# Phase 33: 粒子效果系统

## 概述

原项目包含丰富的粒子效果资源，包括攻击特效、Buff特效、死亡特效等。本阶段实现Godot的粒子系统。

## 特效资源分析

### 1. 攻击特效 (effect/

| 特效类型 | 说明 | 动画帧 |
|---------|------|--------|
| **hitting/ | 攻击命中特效 | 多个 |
| fire/ | 火焰特效 | 13 |
| dark/ | 暗黑特效 | 9 + glow + bg |
| water/ | 水波特效 | 9 + glow + bg |
| light/ | 圣光特效 | 9 + bg |
| magic/ | 魔法特效 | 5 |
| smallSlash1-3 | 小型斩击特效 | 各3 |
| largeSlash1-3 | 大型斩击特效 | 各3 |
| smallKnock/ | 小型击退特效 | 3 |
| largeKnock/ | 大型击退特效 | 3 |
| guard/ | 格挡特效 | 5 |
| heal/ | 治疗特效 | 12 |
| blood/ | 流血特效 | 7 |
| break/ | 破坏特效 | 5 |
| counter/ | 反击特效 | 4 |
| explosion/ | 爆炸特效 | 3 |
| **lastStrike/ | 最终打击特效 | 5 |
| **counterattack1-2 | 反击特效1-2 | 6, 5 |
| **death/ | 死亡特效 |
| normal/ | 普通死亡特效 | 12 |
| flash.png | 闪光特效 | 1 |
| **summon/ | 召唤特效 |
| back/ | 背景召唤特效 | 20 |
| bottom/ | 底部召唤特效 | 20 |
| front/ | 前方召唤特效 | 20 |
| **mark/ | 标记特效 |
| large/ | 大标记特效 | 10 |
| small/ | 小标记特效 | 15 |

### 2. Buff特效 (effect/buff/)

| 特效类型 | 说明 | 动画帧 |
|---------|------|--------|
| bleed/ | 流血特效 | 5 |
| faint1-2/ | 眩晕特效 | 4, 4 |
| confuse1-2/ | 混乱特效 | 4, 4 |
| fear/ | 恐惧特效 | 15 |
| freeze/ | 冰冻特效 | 23 |
| frenzy/ | 狂暴特效 | 8 |
| haste/ | 加速特效 | 12 |
| slow/ | 减速特效 | 10 |
| blind.png | 致盲特效 | 1 |
| poison.png | 中毒特效 | 1 |
| phyAtkUp/Down.png | 物攻增减特效 | 各1 |
| magAtkUp/Down.png | 魔攻增减特效 | 各1 |
| phyDefUp/Down.png | 物防增减特效 | 各1 |
| superArmor.png | 超级护甲特效 | 1 |
| undebuff.png | 解除Debuff特效 | 1 |

### 3. 光环特效 (effect/aura/)

| 特效类型 | 说明 |
|---------|------|
| player.png | 玩家光环特效 |
| partner.png | 伙伴光环特效 |
| named.png | 命名角色光环特效 |
| boss.png | Boss光环特效 |

### 4. 天气特效 (effect/weather/)

| 特效类型 | 说明 | 动画帧 |
|---------|------|
| ray/ | 光线特效 | 6 |
| wall/ | 墙壁特效 | 左, 右 |

## Godot实现方案

### 粒子系统架构

```
EffectManager (Autoload)
├── EffectPool: ParticlePool
├── ├── EffectBase
│   ├── ├── HitEffect
│   ├── ├── BuffEffect
│   ├── ├── DeathEffect
│   ├── ├── SummonEffect
│   ├── └── WeatherEffect
```

### 1. EffectManager (scripts/effect_manager.gd)

```gdscript
class_name EffectManager
extends Node

var effect_scenes: Dictionary = {}
var effect_instances: Array = []

func _ready():
    load_effect_scenes()
    print("EffectManager initialized")

func load_effect_scenes():
    # 加载攻击特效
    effect_scenes["hit_fire"] = load("res://scenes/effects/hit_fire.tscn")
    effect_scenes["hit_dark"] = load("res://scenes/effects/hit_dark.tscn")
    # 加载Buff特效
    effect_scenes["buff_bleed"] = load("res://scenes/effects/buff_bleed.tscn")
    effect_scenes["buff_fear"] = load("res://scenes/effects/buff_fear.tscn")
    print("Effect scenes loaded")

func play_effect(effect_name: String, position: Vector2, parent: Node = null):
    if not effect_scenes.has(effect_name):
        return null
    
    var effect_scene: PackedScene = effect_scenes[effect_name]
    var effect_instance: Node2D = effect_scene.instantiate()
    
    effect_instance.position = position
    
    if parent:
        parent.add_child(effect_instance)
    else:
        get_tree().current_scene.add_child(effect_instance)
    
    effect_instances.append(effect_instance)
    
    # 自动移除
    await effect_instance.finished.connect(func():
        if is_instance_valid(effect_instance):
            effect_instance.queue_free()
            effect_instances.erase(effect_instance))
    
    return effect_instance

func play_hit_effect(hit_type: String, position: Vector2, target_node: Node = null):
    return play_effect("hit_" + hit_type, position, target_node)

func play_buff_effect(buff_type: String, target: Node):
    var effect = play_effect("buff_" + buff_type, target.position, target)
    if effect:
        # 跟随目标
        effect.target_node = target

func play_death_effect(position: Vector2):
    return play_effect("death_normal", position)

func play_summon_effect(position: Vector2, layer: String = "front"):
    return play_effect("summon_" + layer, position)

func play_weather_effect(weather_type: String):
    var effect = play_effect("weather_" + weather_type, Vector2.ZERO)
    if effect:
        effect.z_index = -2

func clear_all_effects():
    for effect in effect_instances:
        if is_instance_valid(effect):
            effect.queue_free()
    effect_instances.clear()
```

### 2. 基础特效场景模板 (scenes/effects/effect_base.tscn)

```gdscript
[gd_scene format=3]

[node name="EffectBase" type="Node2D"]

[node name="Sprite2D" type="Sprite2D" parent="."]
texture = null

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]
```

### 3. 特效基础脚本 (scripts/effect_base.gd)

```gdscript
class_name EffectBase
extends Node2D

signal finished

var sprite: Sprite2D
var animation_player: AnimationPlayer
var frames: SpriteFrames
var is_playing: bool = false

func _ready():
    sprite = $Sprite2D
    animation_player = $AnimationPlayer
    print("EffectBase initialized")

func play():
    is_playing = true
    if animation_player:
        animation_player.play("default")
    elif frames and sprite:
        sprite.play("default")

func stop():
    is_playing = false
    if animation_player:
        animation_player.stop()
    elif frames and sprite:
        sprite.stop()

func on_animation_finished(anim_name: String):
    finished.emit()
```

### 4. 特效场景示例

#### 火焰攻击特效 (scenes/effects/hit_fire.tscn)

```gdscript
[gd_scene format=3]

[ext_resource type="Script" path="res://scripts/effect_base.gd" id="1_effect"]

[node name="HitFire" type="Node2D"]
script = ExtResource("1_effect")

[node name="Sprite2D" type="Sprite2D" parent="."]

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]

[ext_resource type="SpriteFrames" path="res://scenes/effects/hit_fire_sprite_frames.tres" id="2_frames"]
```

### 5. 游戏管理器集成 (game_manager.gd 更新)

```gdscript
# 在顶部添加
var effect_manager: EffectManager

func _ready():
    # ... 现有代码 ...
    setup_effect_manager()

func setup_effect_manager():
    effect_manager = EffectManager.new()
    effect_manager.name = "EffectManager"
    get_tree().root.add_child(effect_manager)
    print("EffectManager setup")

func play_hit_effect(type: String, pos: Vector2, target: Node = null):
    if effect_manager:
        effect_manager.play_hit_effect(type, pos, target)

func play_buff_effect(type: String, target: Node):
    if effect_manager:
        effect_manager.play_buff_effect(type, target)
```

## 实现步骤

### 步骤 1: 创建特效资源目录

```bash
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/scenes/effects
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/scripts/effects
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects
```

### 步骤 2: 复制特效资源

```bash
# 攻击特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/hitting/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/hitting/

# Buff特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/buff/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/buff/

# 死亡特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/death/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/death/

# 召唤特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/summon/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/summon/

# 天气特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/weather/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/weather/
```

### 步骤 3: 创建特效系统文件

1. 特效管理器 (scripts/effect_manager.gd)
2. 特效基础类 (scripts/effect_base.gd)
3. 特效场景模板 (scenes/effects/effect_base.tscn)
4. 具体特效场景 (hit_fire.tscn, hit_dark.tscn 等)

### 步骤 4: 集成到游戏管理器

在 game_manager.gd 中添加特效系统初始化和调用

## 特效使用示例

```gdscript
# 玩家攻击时播放火焰特效
func on_player_attack():
    var hit_pos = player.position + Vector2(50, 0)
    play_hit_effect("fire", hit_pos)

# 敌人受到治疗特效
func on_enemy_healed(enemy: Node):
    play_buff_effect("heal", enemy)

# 玩家死亡特效
func on_player_death():
    play_death_effect(player.position)
```

## 性能优化

1. **对象池**
   - 预加载常用特效
   - 复用而非重创建

2. **纹理图集**
   - 合并相似特效到图集

3. **层级管理**
   - 特效层级 z_index 优化

## 后续改进

- ✅ Phase 34: 粒子效果系统
- ✅ Phase 35: 完整粒子效果增强
- ✅ Phase 36: 粒子特效优化

## 相关文档

- effect_manager_gd.md - 特效管理器代码
- effect_base_gd.md - 特效基础类代码
- effect_scene_template.md - 特效场景模板
