# Phase 28: 精灵动画系统

## 概述

本阶段实现玩家和敌人的精灵动画系统，包括待机、行走、攻击、受伤等动画状态。

## 完成任务

### 1. 动画系统设计
- **状态**: ⏳ 待实现
- **目标**: 为玩家和敌人添加帧动画

### 2. 待完成任务

#### 步骤 1: 准备精灵图集

玩家精灵图应包含以下动画帧：
- idle (待机): 4帧
- walk (行走): 6帧
- attack (攻击): 4帧
- dash (冲刺): 3帧
- hit (受伤): 2帧
- jump (跳跃): 3帧

敌人精灵图应包含以下动画帧：
- idle (待机): 4帧
- walk (行走): 6帧
- attack (攻击): 4帧
- hit (受伤): 2帧

#### 步骤 2: 创建 SpriteFrames 资源

在 Godot 中创建 SpriteFrames 资源，配置各动画的帧序列。

#### 步骤 3: 修改 player_controller.gd

添加动画播放器和状态切换逻辑：

```gdscript
var animation_player: AnimationPlayer = null
var sprite_frames: SpriteFrames = null

func _ready():
    sprite = $Sprite2D
    animation_player = $AnimationPlayer
    sprite_frames = load("res://asset/sprite_frames/player_frames.tres")
    sprite.frames = sprite_frames
    play_animation("idle")

func play_animation(anim_name: String):
    if animation_player and not animation_player.current_animation == anim_name:
        animation_player.play(anim_name)

func update_animation():
    if is_dashing:
        play_animation("dash")
    elif is_attacking:
        play_animation("attack")
    elif velocity.y != 0:
        play_animation("jump")
    elif velocity.x != 0:
        play_animation("walk")
    else:
        play_animation("idle")
```

#### 步骤 4: 修改 enemy.gd

添加类似的动画逻辑：

```gdscript
var animation_player: AnimationPlayer = null

func _ready():
    sprite = $Sprite2D
    animation_player = $AnimationPlayer
    play_animation("idle")

func update_sprite():
    if not sprite:
        return
    
    if velocity.x != 0:
        sprite.flip_h = velocity.x < 0
    
    match current_state:
        EnemyState.IDLE:
            play_animation("idle")
        EnemyState.PATROL:
        EnemyState.CHASE:
            play_animation("walk")
        EnemyState.ATTACK:
            play_animation("attack")
        EnemyState.HIT:
            play_animation("hit")
```

### 3. 技术说明

| 动画状态 | 帧数 | 速度 | 循环 |
|----------|------|------|------|
| idle | 4 | 8 fps | 是 |
| walk | 6 | 12 fps | 是 |
| attack | 4 | 15 fps | 否 |
| dash | 3 | 18 fps | 否 |
| hit | 2 | 10 fps | 否 |
| jump | 3 | 10 fps | 否 |

### 4. 功能特性

| 特性 | 说明 |
|------|------|
| 帧动画 | 支持多帧序列动画 |
| 状态切换 | 根据游戏状态自动切换动画 |
| 速度控制 | 可配置动画播放速度 |
| 循环控制 | 支持循环和单次播放 |

## 相关文件

- `asset/sprite_frames/player_frames.tres` - 玩家精灵帧资源
- `asset/sprite_frames/enemy_frames.tres` - 敌人精灵帧资源
- `scenes/player.tscn` - 玩家场景（添加 AnimationPlayer）
- `scenes/enemy.tscn` - 敌人场景（添加 AnimationPlayer）
- `scripts/player_controller.gd` - 玩家控制器
- `scripts/enemy.gd` - 敌人控制器

---

*创建日期: 2026-04-28*