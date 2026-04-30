# Swordman 动画系统分析

## 概述

Swordman（剑士）角色使用一套完整的序列帧动画系统，包含 242 张序列帧（0-241）。

## 帧分组结构

根据 `config/actor/avatar/duelist/swordman.cfg` 配置：

```lua
files = {
    {0, 20},      -- 第1组：帧 0-20（21帧）
    {33, 41},     -- 第2组：帧 33-41（9帧）
    {75, 241}     -- 第3组：帧 75-241（167帧）
}
```

### 帧分组说明

| 组 | 帧范围 | 数量 | 用途 |
|---|--------|------|------|
| **第1组** | 0-20 | 21帧 | 攻击动作 |
| **第2组** | 33-41 | 9帧 | 攻击3/反击过渡 |
| **第3组** | 75-241 | 167帧 | 各种动作 |

## 完整动画列表

### 🟢 基础动作

| 动画名 | 帧范围 | 帧数 | 每帧时长 | 循环 | 说明 |
|--------|--------|------|----------|------|------|
| **idle** | 90-95 | 6帧 | 120ms | ✓ | 待机 |
| **walk** | 105-112 | 8帧 | 100ms | ✓ | 走路/移动 |

### 🔵 攻击动作

| 动画名 | 帧范围 | 帧数 | 每帧时长 | 循环 | 说明 |
|--------|--------|------|----------|------|------|
| **attack1** | 0-9 | 10帧 | 50ms | ✗ | 普通攻击1 |
| **attack2** | 10-20 | 11帧 | 50ms | ✗ | 普通攻击2 |
| **attack3** | 33-41 | 9帧 | 50ms | ✗ | 普通攻击3 |
| **counterattack** | 10-16 | 7帧 | 50ms | ✗ | 反击 |

### 🟡 防御/受击

| 动画名 | 帧范围 | 帧数 | 每帧时长 | 循环 | 说明 |
|--------|--------|------|----------|------|------|
| **guard** | 123-124 | 2帧 | 100ms | ✓ | 防御姿势 |
| **damage1** | 96 | 1帧 | - | ✗ | 受击1 |
| **damage2** | 99 | 1帧 | - | ✗ | 受击2 |

### 🟠 倒地/坐姿

| 动画名 | 帧范围 | 帧数 | 每帧时长 | 循环 | 说明 |
|--------|--------|------|----------|------|------|
| **down** | 102 | 1帧 | - | ✗ | 倒地 |
| **sit** | 157 | 1帧 | 150ms | ✗ | 坐下 |

### 🔴 技能

| 动画名 | 帧范围 | 帧数 | 每帧时长 | 循环 | 说明 |
|--------|--------|------|----------|------|------|
| **jumonji** | 194,195,191,192,193,211,212,218,219,220,221 | 11帧 | 25-80ms | ✗ | 拔刀斩 |
| **flashStep1** | 240,241,240,132 | 4帧 | 70ms | ✗ | 闪步1 |
| **flashStep2** | 192,193 | 2帧 | 70ms | ✗ | 闪步2 |
| **flashStep3** | 126,129,132 | 3帧 | 70ms | ✗ | 闪步3 |
| **flashStep4** | 240,157,125 | 3帧 | 70ms | ✗ | 闪步4 |
| **onigiri** | 139-156 | 18帧 | 25-150ms | ✗ | 大崩（猛虎落破） |
| **goreCross** | 10-15,194-200 | 13帧 | 25-50ms | ✗ | 逆鳞刺/穿云刺 |

## 帧配置详情

### attack1（普通攻击1）

```lua
return {
    { sprite = "0", time = 50 },
    { sprite = "1", time = 50 },
    { sprite = "2", time = 50 },
    { sprite = "3", time = 50 },
    { sprite = "4", time = 50 },
    { sprite = "5", time = 50 },
    { sprite = "6", time = 50 },
    { sprite = "7", time = 50 },
    { sprite = "8", time = 50 },
    { sprite = "9", time = 150 }  -- 最后一帧停留更久
}
```

### attack2（普通攻击2）

```lua
return {
    { sprite = "10", time = 50 },
    { sprite = "11", time = 50 },
    { sprite = "12", time = 50 },
    { sprite = "13", time = 50 },
    { sprite = "14", time = 50 },
    { sprite = "15", time = 50 },
    { sprite = "16", time = 50 },
    { sprite = "17", time = 50 },
    { sprite = "18", time = 50 },
    { sprite = "19", time = 50 },
    { sprite = "20", time = 150 }
}
```

### attack3（普通攻击3）

```lua
return {
    { sprite = "33", time = 50 },
    { sprite = "34", time = 50 },
    { sprite = "35", time = 50 },
    { sprite = "36", time = 50 },
    { sprite = "37", time = 50 },
    { sprite = "38", time = 50 },
    { sprite = "39", time = 50 },
    { sprite = "40", time = 50 },
    { sprite = "41", time = 150 }
}
```

### guard（防御）

```lua
return {
    { sprite = "123", time = 100 },
    { sprite = "124", time = 100 }
}
```

### damage1（受击1）

```lua
return {
    { sprite = "96" }  -- 单帧，无 time 表示默认
}
```

### damage2（受击2）

```lua
return {
    { sprite = "99" }
}
```

### down（倒地）

```lua
return {
    { sprite = "102" }
}
```

### sit（坐下）

```lua
return {
    { sprite = "157", time = 150 }
}
```

### jumonji（拔刀斩）

```lua
return {
    { sprite = "194", time = 40 },
    { sprite = "195", time = 40 },
    { sprite = "191", time = 40 },
    { sprite = "192", time = 40 },
    { sprite = "193", time = 40 },
    { sprite = "211", time = 40 },
    { sprite = "212", time = 80 },
    { sprite = "218", time = 40 },
    { sprite = "219", time = 40 },
    { sprite = "220", time = 40 },
    { sprite = "221", time = 60 }
}
```

### onigiri（大崩）

```lua
return {
    { sprite = "139", time = 25 },
    { sprite = "140", time = 25 },
    { sprite = "141", time = 25 },
    { sprite = "142", time = 25 },
    { sprite = "143", time = 25 },
    { sprite = "144", time = 50 },
    { sprite = "145", time = 50 },
    { sprite = "146", time = 50 },
    { sprite = "147", time = 50 },
    { sprite = "148", time = 50 },
    { sprite = "149", time = 50 },
    { sprite = "150", time = 50 },
    { sprite = "151", time = 50 },
    { sprite = "152", time = 50 },
    { sprite = "153", time = 50 },
    { sprite = "154", time = 50 },
    { sprite = "155", time = 50 },
    { sprite = "156", time = 150 }
}
```

## 资源文件结构

### 图片资源

```
dfq/asset/textures/actor/duelist/swordman/skin/default/
├── 0.png ~ 241.png  (共 242 张)
```

### SpriteFrames 资源

```
dfq/asset/frames/swordman_sprites.tres
```

包含所有动画的 SpriteFrames 资源文件。

### 帧配置（偏移量）

```
config/asset/sprite/actor/duelist/swordman/skin/default/
├── 0.cfg ~ 241.cfg  (每帧的偏移量配置)
```

**配置格式**：
```lua
return {
    ox = 32,  -- X偏移量
    oy = 100  -- Y偏移量
}
```

## Avatar 系统

Swordman 支持多部位换装，通过 `avatar.cfg` 配置：

**配置路径**：`config/actor/avatar/duelist/swordman.cfg`

```lua
return {
    files = {
        {0, 20},
        {33, 41},
        {75, 241}
    },
    path = "$A",
    layer = {
        belt = 17,
        belt_b = 11,
        cap = 21,
        coat = 18,
        coat_b = 9,
        eyes = 27.1,
        face = 27,
        hair = 20,
        pants = 15,
        shoes = 14,
        skin = 0,
        weapon = 6.5,
        weapon_b = 27.9
    }
}
```

### 层级说明（layer）

数值越小越在底层（先绘制），数值越大越在顶层（后绘制）。

| 部位 | 层级 | 说明 |
|------|------|------|
| skin | 0 | 皮肤/身体（最底层） |
| weapon_b | 27.9 | 武器B（最顶层） |

## Godot 实现

### 资源位置

```
dfq/asset/textures/actor/duelist/swordman/skin/default/
├── 0.png ~ 241.png  (已导入)
```

### SpriteFrames 创建

已创建 `dfq/asset/frames/swordman_sprites.tres`，包含所有动画。

### 测试场景

```
dfq/scenes/swordman_test.tscn
```

用于测试所有动画的演示场景。

## 帧使用统计

当前 Swordman 动画系统共包含 **242 帧（0-241）**。

### 使用情况统计

| 状态 | 帧数 | 占比 |
|------|------|------|
| **已使用** | 96帧 | 39.7% |
| **未使用** | 146帧 | 60.3% |

### 已使用帧详情

| 帧范围 | 数量 | 用途 |
|--------|------|------|
| **0-9** | 10帧 | attack1（普通攻击1） |
| **10-20** | 11帧 | attack2（普通攻击2）、counterattack（反击）、goreCross（逆鳞刺） |
| **33-41** | 9帧 | attack3（普通攻击3） |
| **90-95** | 6帧 | stay/idle（待机） |
| **96** | 1帧 | damage1（受击1） |
| **99** | 1帧 | damage2（受击2） |
| **100** | 1帧 | damage3（受击3） |
| **101** | 1帧 | damage4（受击4） |
| **102** | 1帧 | down（倒地） |
| **105-112** | 8帧 | move/walk（走路） |
| **123-124** | 2帧 | guard（防御） |
| **125** | 1帧 | flashStep4（闪步4） |
| **126** | 1帧 | flashStep3（闪步3） |
| **129** | 1帧 | flashStep3（闪步3） |
| **132** | 1帧 | flashStep1/3（闪步1/3） |
| **139-156** | 18帧 | onigiri（大崩） |
| **157** | 1帧 | sit（坐下）、flashStep4（闪步4） |
| **176-179** | 4帧 | avatar（换装动画） |
| **191-200** | 10帧 | jumonji（拔刀斩）、goreCross（逆鳞刺）、flashStep2（闪步2） |
| **211-212** | 2帧 | jumonji（拔刀斩） |
| **218-221** | 4帧 | jumonji（拔刀斩） |
| **240-241** | 2帧 | flashStep1/4（闪步1/4） |

### 未使用帧清单（146帧）

| 帧范围 | 数量 | 推测用途 |
|--------|------|----------|
| **21-32** | 12帧 | 预留攻击帧，可能用于连击或特殊攻击 |
| **42-89** | 48帧 | 大量预留帧，用于未实现的技能 |
| **97-98** | 2帧 | 额外受击帧预留 |
| **113-122** | 10帧 | 移动/待机变体、跑步动画或冲刺 |
| **127-128** | 2帧 | 防御变体或格挡动作 |
| **130-131** | 2帧 | 闪步变体或闪避动作 |
| **133-138** | 6帧 | onigiri 准备动作或其他技能 |
| **158-175** | 18帧 | 未使用的中间帧或换装变体 |
| **180-190** | 11帧 | 未使用技能帧（可能为 ghostFlash） |
| **201-210** | 10帧 | 拔刀斩变体或月光斩 |
| **213-217** | 5帧 | 拔刀斩中间帧或蓄力动作 |
| **222-239** | 18帧 | 大招或特殊技能预留（如 tripleSlash、forceWave） |

### 未实现的技能引用

根据状态配置文件，以下技能引用了尚未实现的动画配置：

| 技能名称 | 引用的动画配置 | 说明 |
|----------|--------------|------|
| **tripleSlash** | tripleSlash1/2/3 | 三连斩 |
| **tripleSlash_dragon** | - | 龙形三连斩 |
| **moonlight** | moonlight1/2 | 月光斩 |
| **moonlight_full** | moonlight_full | 满月斩 |
| **ghostFlash** | ghostFlash | 鬼影闪 |
| **ghostFlash_cut** | - | 鬼影闪·斩 |
| **ghostWhip** | ghostWhip | 鬼影鞭 |
| **ghostWhip_split** | ghostWhip_split | 鬼影鞭·分裂 |
| **ghostOrb** | - | 鬼珠 |
| **forceWave** | forceWave | 剑气波 |
| **chargeBurst** | - | 蓄力爆发 |
| **breaker** | - | 破招 |
| **blastBlood** | raise | 血爆 |
| **momentary** | - | 瞬发技能 |
| **momentary_tiny** | momentary2 | 小型瞬发 |
| **upperSlash_ascend** | upperSlash_ascend | 上挑斩·升天 |
| **illusion** | - | 幻影 |
| **jumonji_illusion** | - | 拔刀斩·幻影 |
| **jumonji_bleed** | - | 拔刀斩·流血 |
| **throw** | - | 投掷 |
| **call** | - | 召唤 |

### 帧配置文件说明

当前存在的帧动画配置文件：
```
attack1.cfg, attack2.cfg, attack3.cfg, avatar.cfg, counterattack.cfg
damage1.cfg, damage2.cfg, damage3.cfg, damage4.cfg, down.cfg
flashStep1.cfg, flashStep2.cfg, flashStep3.cfg, flashStep4.cfg
goreCross.cfg, guard.cfg, jumonji.cfg, move.cfg, onigiri.cfg
sit.cfg, stay.cfg
```

**缺失的帧动画配置文件**（对应未使用帧）：
```
tripleSlash1.cfg, tripleSlash2.cfg, tripleSlash3.cfg
moonlight1.cfg, moonlight2.cfg, moonlight_full.cfg
ghostFlash.cfg, ghostWhip.cfg, ghostWhip_split.cfg
forceWave.cfg, momentary2.cfg, upperSlash_ascend.cfg
raise.cfg (blastBlood)
```

---

## 新增动画配置发现

通过深入分析 LÖVE 原项目配置，发现了更多技能动画配置文件：

### 状态配置文件（未全部实现）

| 技能名称 | 动画配置 | 说明 |
|----------|----------|------|
| **tripleSlash** | tripleSlash1/2/3 | 三连斩 |
| **moonlight** | moonlight1/2 | 月光斩 |
| **ghostFlash** | ghostFlash | 鬼影闪 |
| **ghostWhip** | - | 鬼影鞭 |
| **ghostOrb** | - | 鬼珠 |
| **breaker** | - | 破招 |
| **chargeBurst** | - | 蓄力爆发 |
| **forceWave** | - | 剑气波 |
| **throw** | - | 投掷 |
| **call** | - | 召唤 |

### 动画配置文件（已确认存在）

| 配置文件 | 帧范围 | 用途 |
|----------|--------|------|
| `avatar.cfg` | 176-179 | 换装动画 |
| `damage3.cfg` | 100 | 受击3 |
| `damage4.cfg` | 101 | 受击4 |

---

## 动画状态机

```
stay (待机)
    ↓ 按方向键
move (移动)
    ↓ 松开方向键
stay (待机)
    ↓ 按攻击键 (J/K)
attack1 (攻击1)
    ↓ 继续按攻击键
attack2 (攻击2)
    ↓
attack3 (攻击3)
    ↓
stay (返回待机)

stay (待机)
    ↓ 按防御键 (L)
guard (防御)
    ↓ 松开防御键
stay (待机)

stay (待机)
    ↓ 被攻击
damage1/damage2 (受击)
    ↓
stay (待机)

stay (待机)
    ↓ 按下蹲键 (S/↓)
sit (坐下)
    ↓ 松开下蹲键
stay (待机)
```

## 参考资料

- LÖVE 源码：`source/actor/drawable/frameani.lua`
- LÖVE 资源配置：`source/actor/resmgr.lua`
- 状态配置：`config/actor/state/duelist/swordman/*.cfg`
- 动画配置：`config/asset/frameani/actor/duelist/swordman/*.cfg`
- Avatar 配置：`config/actor/avatar/duelist/swordman.cfg`
- 帧配置：`config/asset/sprite/actor/duelist/swordman/skin/default/*.cfg`

---

*最后更新：2026-04-30*
