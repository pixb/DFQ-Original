# 武器跟随测试场景文档

## 概述

本测试场景演示了在 Godot 中实现"位置与动画分离"的武器跟随方案，该方案是 DNF 等商业游戏实现"时装+武器特效"的常用底层逻辑。

## 核心概念

### 位置与动画分离

传统方案中，武器位置和动画混合在一起，难以维护。本方案将两者分离：

- **位置层**：由 `HandMarker` 节点负责"武器在哪里"
- **动画层**：由武器精灵负责"武器怎么动"

## 节点结构

```
Character (CharacterBody2D)
├── Body (AnimatedSprite2D)              # 角色身体动画
├── HandMarker (Marker2D)                # 手部标记点（关键帧动画控制）
│   └── WeaponHolder (Node2D)            # 武器容器（自动跟随 HandMarker）
│       ├── Weapon (AnimatedSprite2D)    # 武器本体
│       └── Weapon_b (AnimatedSprite2D)  # 武器特效层
└── AnimationPlayer                      # 控制 HandMarker 位置和 Body 帧
```

## 工作原理

```
1. AnimationPlayer.play("idle_hand")
       │
       ▼
2. Body.frame 逐帧变化（关键帧动画，update=1）
       │
       ▼
3. HandMarker.position 逐帧跳变（关键帧动画，interp=0，update=0）
       │
       ▼
4. WeaponHolder 自动继承 HandMarker 的位置（层级关系）
       │
       ▼
5. Weapon 和 Weapon_b 独立播放特效动画
```

## 场景文件

**位置**：`scenes/weapon_follow_test.tscn`

### 关键配置

#### 1. AnimationPlayer 动画轨道

动画 `idle_hand` 包含两个轨道：

**轨道 0: HandMarker.position**
- **插值方式**：`interp = 0`（最近邻，直接跳变）
- **更新模式**：`update = 0`（连续更新）
- **关键帧**：

| 时间 | 位置 | 说明 |
|------|------|------|
| 0.0s | Vector2(28, 88) | 帧 0 |
| 1.0s | Vector2(28, 87.45) | 帧 1 |
| 2.0s | Vector2(28, 88.43) | 帧 2 |
| 3.0s | Vector2(28, 86.49) | 帧 3 |
| 4.0s | Vector2(28, 86.96) | 帧 4 |
| 5.0s | Vector2(28, 87.48) | 帧 5 |

**轨道 1: Body.frame**
- **插值方式**：`interp = 0`（最近邻）
- **更新模式**：`update = 1`（离散更新，只在关键帧更新）
- **关键帧**：对应时间点设置帧索引 0-5

动画时长：6.0 秒
循环模式：启用

#### 2. 武器精灵配置

| 属性 | Weapon | Weapon_b |
|------|--------|----------|
| z_index | 2 | 5 |
| position | (-25, 30) | (3, 9) |
| offset | (-5, -38) | (-2, 11) |

## 脚本文件

**位置**：`scripts/weapon_follow_test.gd`

### 核心逻辑

```gdscript
extends Node2D

@onready var body_sprite = $Character/Body
@onready var weapon_holder = $Character/HandMarker/WeaponHolder
@onready var weapon_sprite = $Character/HandMarker/WeaponHolder/Weapon
@onready var weapon_b_sprite = $Character/HandMarker/WeaponHolder/Weapon_b
@onready var animation_player = $AnimationPlayer

func _ready():
    weapon_sprite.visible = true
    weapon_b_sprite.visible = true

    body_sprite.play("idle")
    weapon_sprite.play("idle")
    weapon_b_sprite.play("idle")

    print("=== 武器跟随测试场景初始化 ===")
    print("使用节点层级自动跟随，无需代码同步位置")
```

## 关键技术点

### 1. 节点层级跟随（推荐方案）

**为什么这样做？**
- 无需代码同步位置
- 零性能开销
- 自动继承位置、旋转、缩放

**实现方式**：
将 `WeaponHolder` 作为 `HandMarker` 的直接子节点，利用 Godot 的变换继承机制。

### 2. 动画插值方式

| interp 值 | 说明 | 适用场景 |
|-----------|------|----------|
| 0 | Nearest（最近邻） | 像素风格、帧同步动画 |
| 1 | Linear（线性） | 平滑过渡动画 |
| 2 | Cubic（三次） | 流畅曲线动画 |

**本方案选择**：`interp = 0`，确保武器位置与 sprite 帧同步跳变。

### 3. 动画更新模式

| update 值 | 说明 | 适用场景 |
|-----------|------|----------|
| 0 | Continuous（连续） | 每帧都更新值 |
| 1 | Discrete（离散） | 只在关键帧更新值 |

**本方案**：
- `HandMarker.position`：`update = 0`（连续更新，确保武器位置实时）
- `Body.frame`：`update = 1`（离散更新，只在关键帧切换帧）

## 在编辑器中修改动画插值

1. 打开场景 `weapon_follow_test.tscn`
2. 选中 `AnimationPlayer` 节点
3. 打开 Animation 面板（Shift + F6）
4. 选择 `idle_hand` 动画
5. 在轨道列表中找到 `Character/HandMarker:position`
6. 点击轨道右侧的设置图标
7. 将 **Interpolation** 从 `Linear` 改成 `Nearest`

## 测试方法

1. 运行场景 `scenes/weapon_follow_test.tscn`
2. 观察效果：
   - 角色播放 idle 呼吸动画
   - 武器跟随手部位置逐帧跳变（无平滑过渡）
   - 武器同时播放自己的特效动画

## 扩展说明

### 更换武器

通过修改 `WeaponHolder` 下的精灵资源即可更换武器：

```gdscript
weapon_sprite.sprite_frames = new_weapon_frames
weapon_sprite.offset = new_weapon_offset
```

### 添加新动画

1. 在 `AnimationPlayer` 中创建新动画（如 `walk_hand`）
2. 为 `HandMarker.position` 设置新的关键帧（使用 `interp = 0`）
3. 为 `Body.frame` 设置对应的关键帧（使用 `update = 1`）

## 方案演进历史

### 版本 1.0（旧方案）
- 使用 `RemoteTransform2D` 同步位置
- 使用 `_process()` 手动同步
- 存在一帧延迟问题
- 动画插值为线性平滑过渡

### 版本 2.0（当前方案）
- 使用节点层级自动跟随
- 移除代码同步逻辑
- 使用 `interp = 0` 实现帧同步跳变
- 零延迟、零开销

## 优势总结

| 优势 | 说明 |
|------|------|
| 位置与动画分离 | 武器位置和特效动画互不干扰 |
| 零代码同步 | 使用层级关系自动跟随 |
| 帧同步跳变 | `interp = 0` 确保与 sprite 帧同步 |
| 性能优秀 | 无额外计算开销 |
| 易于维护 | 更换武器只需修改精灵资源 |
| 扩展性强 | 支持动态换武器、多套动画 |

---

*文档版本: 2.0*  
*最后更新: 2026-05-01*