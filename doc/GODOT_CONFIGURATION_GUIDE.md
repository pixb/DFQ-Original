# Godot项目配置指南 - 使用LÖVE2D真实资源

## 概述

本指南帮助您在Godot编辑器中配置项目以使用LÖVE2D的真实图片资源。

## 配置步骤

### 步骤 1: 创建SpriteFrames资源

#### 1.1 创建player_sprites.tres

1. 在Godot编辑器中，右键点击 `asset/frames/` 文件夹
2. 选择"新建" → "Resource"
3. 选择"SpriteFrames"
4. 保存为 `player_sprites.tres`

#### 1.2 添加动画帧

1. 打开 `player_sprites.tres`
2. 在"Animations"面板中点击"新建动画"
3. 命名为"idle"
4. 从文件系统面板拖拽以下文件到帧列表：
   - `asset/textures/actor/duelist/swordman/skin/default/0.png`
   - `asset/textures/actor/duelist/swordman/skin/default/1.png`
   - `asset/textures/actor/duelist/swordman/skin/default/2.png`
   - `asset/textures/actor/duelist/swordman/skin/default/3.png`
5. 设置速度为5.0，勾选"Loop"
6. 重复以上步骤创建"walk"动画（速度8.0）

### 步骤 2: 配置Player场景

#### 2.1 更新player.tscn

1. 打开 `scenes/player.tscn`
2. 选择AnimatedSprite2D节点
3. 在检查器中设置：
   - Frames: `asset/frames/player_sprites.tres`
   - Animation: "idle"
   - Playing: true

#### 2.2 确保节点结构

```
Player (CharacterBody2D)
├── CollisionShape2D
├── AnimatedSprite2D
└── Camera2D
```

### 步骤 3: 配置视差背景

#### 3.1 更新parallax_background.tscn

文件已更新为使用真实纹理：
- FarLayer: `asset/textures/map/lorien/far.png` (parallax_scale: 0.3)
- NearLayer: `asset/textures/map/lorien/near.png` (parallax_scale: 0.7)

#### 3.2 验证纹理加载

1. 运行项目
2. 检查调试输出是否显示：
   - "Parallax background loaded"
   - 无"Could not load"错误

### 步骤 4: 配置输入映射

#### 4.1 打开项目设置

1. 点击"项目" → "项目设置"
2. 选择"输入映射"选项卡

#### 4.2 添加自定义动作

添加以下动作（如果不存在）：

| 动作名 | 建议按键 |
|--------|----------|
| left | A / Left Arrow |
| right | D / Right Arrow |
| up | W / Up Arrow |
| down | S / Down Arrow |
| jump | Space |
| dash | Shift |
| attack | J / K |

#### 4.3 使用默认UI动作

或者直接使用Godot默认的UI动作：
- `ui_left`, `ui_right`, `ui_up`, `ui_down`
- `ui_accept` (Space)
- `ui_cancel` (Escape)
- `ui_select` (Shift)
- `ui_focus_next` (Tab)

### 步骤 5: 配置敌人精灵

#### 5.1 创建goblin_sprites.tres

1. 创建新的SpriteFrames资源
2. 添加动画：idle, walk, attack, hurt
3. 从 `asset/textures/actor/duelist/goblin/` 加载帧

#### 5.2 更新enemy.tscn

1. 打开 `scenes/enemy.tscn`
2. 选择Sprite2D或AnimatedSprite2D节点
3. 设置Frames或直接设置Texture

### 步骤 6: 配置装饰系统

#### 6.1 更新decorator_manager.gd

文件已创建，使用以下资源：

```gdscript
# 树木 (parallax_rate: 0.4)
res://asset/textures/map/lorien/tree/0.png
res://asset/textures/map/lorien/tree/1.png
res://asset/textures/map/lorien/tree/2.png
...

# 石头 (parallax_rate: 0.6)
res://asset/textures/map/lorien/stone/0.png
...

# 花朵 (parallax_rate: 0.8)
res://asset/textures/map/lorien/flower/0.png
...

# 草地 (parallax_rate: 0.9)
res://asset/textures/map/lorien/grass/0.png
...
```

### 步骤 7: 测试运行

#### 7.1 运行测试

1. 按F5或点击运行按钮
2. 检查输出面板
3. 确认无错误

#### 7.2 预期输出

```
Godot Engine started
Game started!
Parallax background loaded
Player sprite initialized with real textures
Player spawned at: (640.0, 360.0)
Enemy sprite initialized
Spawned 3 enemies
Background music started: lorien.mp3
```

## 资源路径参考

### 地图资源

| 资源 | Godot路径 |
|------|-----------|
| 远景背景 | res://asset/textures/map/lorien/far.png |
| 近景背景 | res://asset/textures/map/lorien/near.png |
| 树木 | res://asset/textures/map/lorien/tree/X.png |
| 石头 | res://asset/textures/map/lorien/stone/X.png |
| 花朵 | res://asset/textures/map/lorien/flower/X.png |
| 草地 | res://asset/textures/map/lorien/grass/X.png |

### 角色资源

| 角色 | Godot路径 |
|------|-----------|
| Swordman | res://asset/textures/actor/duelist/swordman/skin/default/X.png |
| Goblin | res://asset/textures/actor/duelist/goblin/skin/X.png |
| Tau | res://asset/textures/actor/duelist/tau/skin/X.png |

### 特效资源

| 类型 | Godot路径 |
|------|-----------|
| 攻击特效 | res://asset/textures/effects/hitting/fire/X.png |
| Buff特效 | res://asset/textures/effects/buff/bleed/X.png |
| 死亡特效 | res://asset/textures/effects/death/normal/X.png |

## 常见问题

### Q: 纹理显示为粉色/紫色？
A: 纹理路径错误或文件损坏，检查控制台错误信息。

### Q: 动画不播放？
A: 检查SpriteFrames配置，确保动画名称正确。

### Q: 输入不响应？
A: 检查输入映射配置，或使用默认UI动作。

### Q: 资源加载失败？
A: 确保资源文件已复制到Godot项目目录，且路径正确。

## 下一步

配置完成后，您可以：
1. 调整视差滚动的速度比例
2. 添加更多装饰元素
3. 实现粒子特效系统
4. 优化性能

---

*创建日期: 2026-04-29*
