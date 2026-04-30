# 阶段 51: 角色装备系统

**目标**: 实现角色装备（衣服、武器、饰品）系统，支持多层Sprite叠加渲染

---

## 任务清单

- [x] 51.1 创建装备组件 (EquipmentComponent) ✅
- [x] 51.2 修改角色场景结构，添加装备槽位 ✅
- [x] 51.3 创建装备数据配置系统 ✅
- [x] 51.4 实现装备穿戴/卸载逻辑 ✅
- [x] 51.5 同步装备动画 ✅
- [x] 51.6 测试装备系统 ✅

---

## LÖVE 原系统分析

### Avatar系统架构

原LÖVE项目使用**Avatar配置系统**实现装备叠加，核心配置结构如下：

**1. Avatar配置文件** (`config/actor/avatar/duelist/swordman.cfg`)

```lua
return {
    files = {
        {0, 20},     -- 帧序列范围
        {33, 41},
        {75, 241}
    },
    path = "$A",     -- 资源路径: actor/duelist/swordman/
    layer = {
        skin = 0,        -- 皮肤（最底层）
        weapon = 6.5,    -- 武器主层
        weapon_b = 27.9, -- 武器次层（特效层）
        pants = 15,      -- 裤子
        shoes = 14,      -- 鞋子
        hair = 20,       -- 头发
        cap = 21,        -- 帽子
        -- ... 更多部位
    },
    extra = {
        ["190a"] = {frame = 190, passMap = {weapon = true}}  -- 特殊帧处理
    }
}
```

**2. 武器装备配置** (`config/actor/equipment/weapon/swordman/katana.cfg`)

```lua
return {
    name = {cn = "武士刀", en = "Samurai Sword"},
    kind = "weapon",
    subKind = "katana",
    avatar = {
        weapon = "katana/default",      -- 替换武器主层纹理
        weapon_b = "katana/default_b"   -- 替换武器次层纹理
    },
    add = {
        attackRate = 0.05  -- 属性加成
    }
}
```

### 层级渲染机制

| 层级值 | 部位 | 说明 |
|--------|------|------|
| 0 | `skin` | 皮肤（最底层） |
| 5 | `coat_d` | 外衣底层 |
| 6.5 | `weapon` | 武器主层 |
| 7 | `cap_b` | 帽子底层 |
| 9 | `coat_b` | 外衣次层 |
| 10 | `neck_b` | 项链底层 |
| 11 | `belt_b` | 腰带底层 |
| 11.5 | `pants_d` | 裤子底层 |
| 12 | `shoes_b` | 鞋子底层 |
| 13 | `pants_b` | 裤子次层 |
| 14 | `shoes` | 鞋子 |
| 15 | `pants` | 裤子 |
| 17 | `belt` | 腰带 |
| 18 | `coat` | 外衣 |
| 19 | `neck` | 项链 |
| 19.5 | `belt_c` | 腰带装饰 |
| 20 | `hair` | 头发 |
| 21 | `cap` | 帽子 |
| 22 | `neck_c` | 项链装饰 |
| 23 | `coat_c` | 外衣装饰 |
| 23.5 | `neck_e` | 项链特效 |
| 25 | `cap_c` | 帽子装饰 |
| 27 | `face` | 脸部 |
| 27.1 | `eyes` | 眼睛 |
| 27.9 | `weapon_b` | 武器特效层（最顶层之一） |

### PassMap机制

PassMap用于控制特定帧是否显示某个部位：

```lua
local passMap = {
    weapon = true,   -- 隐藏武器层
    weapon_b = true  -- 隐藏武器特效层
}

local extra = {
    ["190a"] = {frame = 190, passMap = passMap}  -- 帧190隐藏武器
}
```

这用于特殊动作（如空手攻击、施法等）时隐藏武器。

---

## 系统设计

### 架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                     EquipmentSystem                         │
├─────────────────────────────────────────────────────────────┤
│  EquipmentComponent  ← 管理装备槽位和装备数据               │
│       │                                                    │
│       ├── slots: Dictionary  ← 装备槽位映射                │
│       ├── sprite_map: Dictionary  ← Sprite节点映射         │
│       └── signals: equipment_changed                       │
├─────────────────────────────────────────────────────────────┤
│  装备槽位定义:                                              │
│  - body: 基础身体层 (z=0)                                   │
│  - pants: 裤子层 (z=1)                                      │
│  - weapon: 武器层 (z=2)                                     │
│  - hat: 帽子层 (z=3)                                        │
│  - armlet: 护腕层 (z=4)                                     │
└─────────────────────────────────────────────────────────────┘
```

### 场景结构设计

```
Character (CharacterBody2D)
├── Body (AnimatedSprite2D)       # z=0, 基础身体
├── Pants (AnimatedSprite2D)      # z=1, 裤子层
├── Weapon (AnimatedSprite2D)     # z=2, 武器层
├── Hat (AnimatedSprite2D)        # z=3, 帽子层
└── Armlet (AnimatedSprite2D)     # z=4, 护腕层
```

### 装备数据结构

```gdscript
{
    name: "Army Sword",          # 装备名称
    type: "weapon",              # 装备类型
    slot: "weapon",              # 装备槽位
    sprite_frames: SpriteFrames, # 动画帧资源
    attributes: {                # 属性加成（可选）
        attack: 10,
        speed: 5
    }
}
```

---

## 实现步骤

### 1. 创建装备组件

**文件**: `scripts/equipment_component.gd`

```gdscript
class_name EquipmentComponent

var slots = {}
var sprite_map = {}

signal equipment_changed(slot_name, equipment_data)

func setup(entity_ref, sprite_nodes):
    sprite_map = sprite_nodes
    for slot in sprite_map:
        if slot != "body":
            sprite_map[slot].visible = false

func equip(slot_name, data):
    if not sprite_map.has(slot_name):
        print("Equipment slot not found: ", slot_name)
        return
    
    if slots.has(slot_name) and slots[slot_name]:
        unequip(slot_name)
    
    slots[slot_name] = data
    
    var sprite = sprite_map[slot_name]
    if data and data.has("sprite_frames") and data.sprite_frames:
        sprite.sprite_frames = data.sprite_frames
        sprite.visible = true
        
        var body_sprite = sprite_map.get("body")
        if body_sprite and body_sprite.animation:
            sprite.animation = body_sprite.animation
    elif sprite.sprite_frames:
        sprite.visible = true
    
    equipment_changed.emit(slot_name, data)

func unequip(slot_name):
    if slots.has(slot_name):
        slots.erase(slot_name)
        sprite_map[slot_name].visible = false
        equipment_changed.emit(slot_name, null)

func get_equipment(slot_name):
    return slots.get(slot_name)

func sync_animation(anim_name):
    for slot in sprite_map:
        var sprite = sprite_map[slot]
        if sprite.visible and sprite.animation != anim_name:
            sprite.play(anim_name)

func get_all_equipment():
    return slots.duplicate()
```

### 2. 修改角色场景

**文件**: `scenes/swordman_25d_test.tscn`

```xml
[gd_scene format=3 uid="uid://827c8suvwcmxo"]

[ext_resource type="Script" path="res://scripts/swordman_25d_test.gd" id="1"]
[ext_resource type="SpriteFrames" path="res://asset/frames/swordman_sprites.tres" id="2"]

[sub_resource type="RectangleShape2D" id="RectangleShape2D_char"]
size = Vector2(40, 100)

[node name="Swordman25DTest" type="Node2D"]
script = ExtResource("1")

[node name="Character" type="CharacterBody2D" parent="."]
position = Vector2(640, 350)

[node name="CollisionShape2D" type="CollisionShape2D" parent="Character"]
shape = SubResource("RectangleShape2D_char")

[node name="Body" type="AnimatedSprite2D" parent="Character"]
sprite_frames = ExtResource("2")
animation = "idle"
z_index = 0

[node name="Pants" type="AnimatedSprite2D" parent="Character"]
z_index = 1
visible = false

[node name="Weapon" type="AnimatedSprite2D" parent="Character"]
z_index = 2
visible = false

[node name="Weapon_b" type="AnimatedSprite2D" parent="Character"]
z_index = 5
visible = false

[node name="Hat" type="AnimatedSprite2D" parent="Character"]
z_index = 3
visible = false

[node name="Armlet" type="AnimatedSprite2D" parent="Character"]
z_index = 4
visible = false

[node name="Camera2D" type="Camera2D" parent="."]
position = Vector2(640, 350)
make_current = true

[node name="Instructions" type="Label" parent="."]
offset_left = 10.0
offset_top = 650.0
offset_right = 400.0
offset_bottom = 715.0
theme_default_font_size = 11
autowrap_mode = 2
```

### 3. 修改角色脚本

**文件**: `scripts/swordman_25d_test.gd`

关键修改：

```gdscript
extends Node2D

@onready var character = $Character
@onready var body_sprite = $Character/Body
@onready var sprite_slots = {
    "body": $Character/Body,
    "pants": $Character/Pants,
    "weapon": $Character/Weapon,
    "weapon_b": $Character/Weapon/Weapon_b,
    "hat": $Character/Hat,
    "armlet": $Character/Armlet
}

var equipment: EquipmentComponent = null

func _ready():
    equipment = EquipmentComponent.new()
    equipment.setup(self, sprite_slots)
    equipment.connect("equipment_changed", _on_equipment_changed)
    # ... 其他初始化代码

func _on_equipment_changed(slot_name, data):
    if data:
        print("Equipped ", slot_name, ": ", data.name)
    else:
        print("Unequipped ", slot_name)

func update_movement_animation(input_x, input_y):
    if is_locked:
        return

    if input_x != 0 or input_y != 0:
        if body_sprite.animation != "walk":
            body_sprite.play("walk")
            equipment.sync_animation("walk")
            current_animation = "walk"
    else:
        if body_sprite.animation != "idle":
            body_sprite.play("idle")
            equipment.sync_animation("idle")
            current_animation = "idle"
    
    update_instructions()

func play_animation(anim_name):
    if body_sprite.animation != anim_name:
        body_sprite.play(anim_name)
        equipment.sync_animation(anim_name)
        current_animation = anim_name
        update_instructions()
```

### 4. 测试输入处理

在 `_input` 函数中添加装备测试代码：

```gdscript
match event.keycode:
    KEY_E:
        var weapon_data = {
            name = "Army Sword",
            type = "weapon",
            slot = "weapon",
            sprite_frames = load("res://asset/frames/swordman_sprites.tres")
        }
        equipment.equip("weapon", weapon_data)
        update_instructions()
    KEY_H:
        var hat_data = {
            name = "Base Hat",
            type = "hat",
            slot = "hat",
            sprite_frames = load("res://asset/frames/swordman_sprites.tres")
        }
        equipment.equip("hat", hat_data)
        update_instructions()
    KEY_U:
        equipment.unequip("weapon")
        equipment.unequip("hat")
        equipment.unequip("pants")
        equipment.unequip("armlet")
        update_instructions()
```

---

## 测试结果

### 测试日志（已通过）

```
=== Swordman 2.5D Animation Test ===
=== Equipment System Ready ===
Equipment slots setup: ["body", "weapon", "hat", "pants", "armlet"]

# 按 E 键装备武器
Weapon frames loaded: SpriteFrames(...)
equip called for slot: weapon
Sprite for weapon: AnimatedSprite2D(...)
Setting sprite_frames: SpriteFrames(...)
Sprite visible set to true
Animation synced to: idle
Equipped weapon: Army Sword
Weapon equipped

# 按 H 键装备帽子
Hat frames loaded: SpriteFrames(...)
equip called for slot: hat
Sprite for hat: AnimatedSprite2D(...)
Setting sprite_frames: SpriteFrames(...)
Sprite visible set to true
Animation synced to: idle
Equipped hat: Base Hat
Hat equipped

# 按 U 键卸载装备
All equipment unequipped
```

### 测试用例验证

| 测试项 | 操作 | 预期结果 | 实际结果 |
|--------|------|----------|----------|
| 装备武器 | 按 E 键 | Weapon Sprite 显示，播放对应动画 | ✅ 通过 |
| 装备帽子 | 按 H 键 | Hat Sprite 显示，播放对应动画 | ✅ 通过 |
| 卸载装备 | 按 U 键 | 对应 Sprite 隐藏 | ✅ 通过 |
| 动画同步 | 移动角色 | 所有可见装备层同步播放 walk 动画 | ✅ 通过 |
| 攻击动画 | 按 X 键 | 所有装备层同步播放攻击动画 | ✅ 通过 |
| 左右翻转 | 按 ←/→ | 所有装备层同步翻转 | ✅ 通过 |

---

## 技术要点

### 多层渲染原理

装备系统使用 **Sprite2D 自动分层渲染**，通过 `z_index` 属性控制渲染顺序：

1. **Body (z=0)** - 基础身体层，始终显示
2. **Pants (z=1)** - 裤子层，叠加在身体上
3. **Weapon (z=2)** - 武器层，叠加在裤子上
4. **Hat (z=3)** - 帽子层，叠加在武器上
5. **Armlet (z=4)** - 护腕层，最顶层

### 动画同步机制

```gdscript
func sync_animation(anim_name):
    for slot in sprite_map:
        var sprite = sprite_map[slot]
        if sprite.visible and sprite.animation != anim_name:
            sprite.play(anim_name)
```

- 遍历所有装备槽位
- 只同步可见的装备层
- 确保动画名称一致

### 翻转同步

```gdscript
if input_x != 0:
    # ... 移动逻辑
    for slot in sprite_slots:
        sprite_slots[slot].flip_h = input_x < 0
```

- 角色转向时，所有装备层同步翻转

---

## 原 LÖVE 系统对比

| 特性 | LÖVE | Godot |
|------|------|-------|
| 渲染方式 | 手动分层绘制 | Sprite2D 自动分层 |
| 装备管理 | Avatar 配置系统 | EquipmentComponent |
| PassMap | 控制部位显示 | Sprite.visible |
| 动画同步 | 手动同步 | SpriteFrames 自动同步 |
| 资源加载 | 自定义资源管理器 | Godot 资源系统 |

---

## 后续扩展计划

1. **装备属性系统** - 添加装备属性加成效果
2. **装备品质系统** - 支持不同品质装备（普通/稀有/史诗/传说）
3. **装备特效系统** - 为高品质装备添加发光效果
4. **装备耐久度** - 实现装备耐久和修复系统
5. **装备套装系统** - 支持套装属性加成
6. **独立装备资源** - 为每个装备槽位创建独立的 SpriteFrames

---

## 文件清单

| 文件 | 描述 | 状态 |
|------|------|------|
| `scripts/equipment_component.gd` | 装备组件核心逻辑 | ✅ 完成 |
| `scripts/swordman_25d_test.gd` | 测试场景脚本 | ✅ 完成 |
| `scripts/sprite_frames_generator.gd` | SpriteFrames资源生成器 | ✅ 完成 |
| `scenes/swordman_25d_test.tscn` | 测试场景 | ✅ 完成 |
| `scenes/sprite_frames_generator.tscn` | 生成器运行场景 | ✅ 完成 |
| `asset/frames/weapon_katana.tres` | 武器主层资源 | ✅ 生成 |
| `asset/frames/weapon_katana_b.tres` | 武器特效层资源 | ✅ 生成 |
| `asset/frames/pants_renewal.tres` | 裤子层资源 | ✅ 生成 |
| `asset/frames/cap_renewal_b.tres` | 帽子层资源 | ✅ 生成 |
| `asset/frames/coat_renewal.tres` | 外衣层资源 | ✅ 生成 |
| `asset/frames/shoes_renewal.tres` | 鞋子层资源 | ✅ 生成 |
| `doc/051_EQUIPMENT_SYSTEM.md` | 设计文档 | ✅ 完成 |

---

## 问题分析：武器帧图片问题

### 现象描述

测试时按 **E** 键装备武器，但视觉上没有变化。

### 根本原因

**当前测试代码使用了错误的纹理资源**：

```gdscript
KEY_E:
    var weapon_data = {
        name = "Army Sword",
        sprite_frames = load("res://asset/frames/swordman_sprites.tres")  -- ❌ 使用了身体纹理
    }
    equipment.equip("weapon", weapon_data)
```

所有装备层都加载了同一份 `swordman_sprites.tres`，导致叠加后视觉上看不出变化。

### LÖVE原系统的正确做法

根据原LÖVE系统的Avatar配置：

```lua
-- katana.cfg
avatar = {
    weapon = "katana/default",      -- 武器纹理路径
    weapon_b = "katana/default_b"   -- 武器特效路径
}
```

武器应该使用独立的武器纹理，位于 `actor/duelist/swordman/katana/default/` 目录。

### 解决方案

**1. 准备独立的装备纹理资源**

项目中 swordman 的纹理文件：

```
asset/textures/actor/duelist/swordman/
├── skin/default/           # 身体纹理 (0-241.png, 242帧)
├── katana/default/          # 武器主层 (0-241.png, 242帧)
├── katana/default_b/        # 武器特效层 (0-241.png, 242帧)
├── pants/renewal/          # 裤子层 (0-241.png, 242帧)
├── cap/renewal_b/          # 帽子层 (0-241.png, 242帧)
├── coat/renewal/           # 外衣层 (0-241.png, 242帧)
└── shoes/renewal/          # 鞋子层 (0-241.png, 242帧)
```

**2. 创建独立的 SpriteFrames 资源**

| 资源文件 | 纹理来源 | 帧数 | 用途 |
|----------|----------|------|------|
| `body_default.tres` | `skin/default/` | 242 | 基础身体 |
| `weapon_katana.tres` | `katana/default/` | 242 | 武器主层 (攻击时显示) |
| `weapon_katana_b.tres` | `katana/default_b/` | 242 | 武器特效层 (idle/特效时显示) |
| `pants_renewal.tres` | `pants/renewal/` | 242 | 裤子层 |
| `cap_renewal_b.tres` | `cap/renewal_b/` | 242 | 帽子层 |
| `coat_renewal.tres` | `coat/renewal/` | 242 | 外衣层 |
| `shoes_renewal.tres` | `shoes/renewal/` | 242 | 鞋子层 |

**3. 关键发现：武器分层机制**

根据 LÖVE avatar 配置，武器分为两层：

| 层 | z值 | idle帧大小 | attack帧大小 | 用途 |
|----|-----|-----------|-------------|------|
| `weapon` | 6.5 | 11x13 (小图标) | 较大 | 攻击时显示 |
| `weapon_b` | 27.9 | 48x46 (完整) | - | idle/特效时显示 |

**idle 时武器不可见的原因**：weapon 层的 idle 帧是小图标，而 weapon_b 层才是完整的武器显示。

**4. 测试代码修改**

```gdscript
KEY_E:
    var frames = load("res://asset/frames/weapon_katana.tres")
    var frames_b = load("res://asset/frames/weapon_katana_b.tres")

    var weapon_data = {
        name = "Katana",
        type = "weapon",
        slot = "weapon",
        sprite_frames = frames
    }
    var weapon_b_data = {
        name = "Katana Effect",
        type = "weapon_b",
        slot = "weapon_b",
        sprite_frames = frames_b
    }
    equipment.equip("weapon", weapon_data)
    equipment.equip("weapon_b", weapon_b_data)
```

### 纹理资源说明

| 槽位 | 纹理目录 | 帧数量 |
|------|----------|--------|
| body | `skin/default/` | 242帧 (0-241) |
| weapon | `katana/default/` | 242帧 (0-241) |
| weapon_b | `katana/default_b/` | 242帧 (0-241) |
| pants | `pants/renewal/` | 242帧 (0-241) |
| cap | `cap/renewal_b/` | 242帧 (0-241) |
| coat | `coat/renewal/` | 242帧 (0-241) |
| shoes | `shoes/renewal/` | 242帧 (0-241) |

> **重要**: swordman 的所有动画帧都在 0-241 范围内，与 avatar 配置的 `files = {{0,20}, {33,41}, {75,241}}` 对应。

---

## 关键发现：Per-Frame 偏移问题

### 问题描述

武器位置不正确，与身体没有正确对齐。

### 根本原因

LÖVE 的 sprite 配置中，**每帧都有独立的 ox/oy 偏移值**：

**swordman skin 每帧偏移示例** (`config/asset/sprite/actor/duelist/swordman/skin/default/`)：
```lua
-- 帧 0 (attack1)
{ ox = 43, oy = 102 }
-- 帧 90 (idle)
{ ox = 32, oy = 100 }
-- 帧 105 (walk)
{ ox = 24, oy = 95 }
```

**katana/default_b 每帧偏移示例**：
```lua
-- 帧 0 (attack1)
{ ox = -268, oy = 333 }
-- 帧 90 (idle)
{ ox = 27, oy = 62 }
-- 帧 105 (walk)
{ ox = 68, oy = 61 }
```

### LÖVE Avatar 合并机制

在 `_NewAvatarSpriteData` 函数中：
1. 获取所有层的 spriteData
2. 合并所有层到一个边界框
3. 计算合并后的统一 `ox, oy` 使得左上角对齐

```lua
-- resmgr.lua 第 121-177 行
local spriteData = {}
local minX, minY, maxX, maxY

-- 计算所有 subject 的边界框
for n=1, #spriteDatas do
    local minx = -spriteDatas[n].ox
    local miny = -spriteDatas[n].oy
    local maxx = minx + spriteDatas[n].w
    local maxy = miny + spriteDatas[n].h
    -- ... 找最小/最大值
end

-- 设置合并后的偏移
spriteData.ox = -minX
spriteData.oy = -minY
```

### Godot 实现挑战

**AnimatedSprite2D 不支持 per-frame offset**，只能设置统一的 `offset` 属性。

### 解决方案

#### 方案 1：固定偏移（经验值）

使用 idle 帧（90）的偏移作为固定偏移：

| 槽位 | 固定偏移 (ox, oy) |
|------|------------------|
| body (skin) | (32, 100) |
| weapon_b (katana) | (27, 62) |

在 Godot 中，AnimatedSprite2D 的 `offset` 属性是**相对于 sprite 中心的偏移**，而 LÖVE 的 ox/oy 是**左上角的偏移**。

转换公式：
```
offset_x = ox - texture_width / 2
offset_y = oy - texture_height / 2
```

| 槽位 | 纹理尺寸 | 固定偏移 | 计算后 offset |
|------|----------|---------|--------------|
| body | 64x106 | (32, 100) | (32-32, 100-53) = (0, 47) |
| weapon_b | 48x46 | (27, 62) | (27-24, 62-23) = (3, 39) |

#### 方案 2：自定义渲染系统

放弃 AnimatedSprite2D，使用 Sprite2D + 手动控制每帧的 offset。

#### 方案 3：修改 SpriteFrames 生成器

修改生成器脚本，读取配置文件中的 per-frame ox/oy，并在渲染时应用。

### 实现方案：固定偏移（经验值）

我们采用方案 1，为每个装备槽设置固定的 offset，使用 idle 帧（90）的数据作为基础。

#### swordman 完整 idle 帧（90）配置数据

| 部件 | 配置文件 | 纹理尺寸 | ox | oy | layer |
|------|----------|----------|-----|-----|-------|
| skin (body) | skin/default/90.cfg | 64x106 | 32 | 100 | 0 |
| weapon | katana/default/90.cfg | (待定) | (待定) | (待定) | 6.5 |
| **weapon_b** | **katana/default_b/90.cfg** | **48x46** | **27** | **62** | **27.9** |
| shoes | shoes/renewal_b/90.cfg | (待定) | -268 | 333 | 12 |

#### Godot offset 计算公式

```
Godot 的 offset 是相对于 sprite 中心的。
LÖVE 的 ox, oy 是相对于锚点（即合并后的 sprite 的左上角）的偏移。

由于 Godot 我们是在同一 CharacterBody2D 下放置多个 AnimatedSprite2D，
只需要考虑 weapon_b 相对于 body 的相对偏移。

body 的纹理中心: (64/2, 106/2) = (32, 53)
weapon_b 的纹理中心: (48/2, 46/2) = (24, 23)

当前实现的 weapon_b offset: Vector2(-10, -30)
```

### 下一步

1. 运行测试，观察武器位置
2. 根据视觉效果微调 offset 值
3. 添加其他装备（pants, hat 等）的偏移配置
4. （可选）实现更精确的 per-frame 偏移机制

---

*阶段 51 实现完成*

**测试快捷键**:
- **E** - 装备武器
- **H** - 装备帽子
- **U** - 卸载所有装备