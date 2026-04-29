# DFQ 渲染差异分析与修复规划

## 概述

对比 LÖVE2D 原版项目和当前 Godot 项目的渲染实现，找出差异并修复。

---

## 核心问题清单

| 问题编号 | 问题描述 | 优先级 | 影响 |
|---------|---------|-------|------|
| R01 | 精灵缩放不正确 | 高 | 角色太大/太小 |
| R02 | 精灵原点 (ox, oy) 未正确设置 | 高 | 角色位置偏移 |
| R03 | 视差背景层顺序不对 | 高 | 深度感缺失 |
| R04 | 装饰系统缺失 (树、石头、花等) | 高 | 地图不完整 |
| R05 | 角色多层渲染缺失 | 高 | 角色不完整 |
| R06 | 相机跟随逻辑不对 | 中 | 地图看起来偏移 |

---

## 详细分析

### 1. LÖVE2D 原项目的渲染流程

#### 核心模块
```
Director.Init() -> 初始化核心
├── Graphics.Init()
├── Map.Init()
│   ├── Background (视差层)
│   └── Decorations (装饰)
└── World.Init()
    └── Actor (角色渲染)
```

#### 地图渲染配置 (lorien.cfg)
```
far = "$A/far"            -- 远景背景
near = "$A/near"          -- 近景背景
floor = {                 -- 地板
    top = "$A/tile/2"
    extra = "$A/tile/3"
    horizon = 327
}
object.up = {             -- 远景装饰
    tree/2, tree/3, ...
}
object.floor = {          -- 近景装饰
    grass, trail, ...
}
```

#### 精灵属性 (原项目关键参数)
| 属性 | 原项目 (LÖVE2D) | 当前 Godot | 状态 |
|-----|----------------|------------|------|
| 缩放 | (sx, sy) | 未正确设置 | ❌ 需修复 |
| 原点 | (ox, oy) | 未设置 | ❌ 需修复 |
| 角度 | angle | 未设置 | ⚠️ 后期 |
| 颜色 | color | 未设置 | ⚠️ 后期 |
| 混合模式 | blendmode | 未设置 | ⚠️ 后期 |

---

### 2. 树/装饰的原点分析

示例：tree/0.cfg
```lua
ox = 141,
oy = 308
```
表示精灵的原点在 (141, 308)，意思是精灵的底部在 Y 轴上靠下的位置，这样树看起来是站在地面上的！

---

### 3. 角色的多层渲染

原项目中角色使用多图层渲染：
- skin
- pants
- coat
- weapon
- 等等共 20+ 层

**当前 Godot 问题**：只用了单一皮肤图层，角色看起来不完整！

---

## 修复规划

### 阶段 1：修复基础精灵配置 (高优先级)

**目标**：让角色和背景的缩放、原点正确

#### 文件清单
- [ ] `scenes/player.tscn` - 添加正确的缩放和原点
- [ ] `scenes/enemy.tscn` - 添加正确的缩放和原点
- [ ] `asset/frames/player_sprites.tres` - 可能需要调整
- [ ] `asset/frames/enemy_sprites.tres` - 可能需要调整

#### 具体配置
```
玩家 (swordman) 估计缩放：1.0 - 2.0 (需要测试)
敌人 (goblin) 估计缩放：1.0 - 2.0 (需要测试)
```

---

### 阶段 2：修复视差背景系统 (高优先级)

**目标**：让视差背景更接近原版

#### 当前 Godot 问题
1. 背景层顺序简单：far, near 两层
2. 没有完整的视差滚动逻辑
3. 缺少中间装饰层

#### LÖVE2D 原版的背景系统
- 使用多个背景层 (far, near)
- 每层有不同的 rate 参数
- 背景跟随相机移动（shift * rate）

#### 修复方案
更新 `scenes/parallax_background.tscn`，可能需要多个 ParallaxLayer！

---

### 阶段 3：实现完整的装饰系统 (高优先级)

**目标**：加入树、石头、花等装饰，完全复现原版地图！

#### 装饰清单
- 树 (tree)：11 个种类
- 石头 (stone)：6 个种类
- 花 (flower)：5 个种类
- 草 (grass)：4 个种类
- 石柱 (stonePillar)：4 个种类
- 小路 (trail)：3 个种类
- 瓦片 (tile)：4 个种类

#### 装饰分布
- **远景装饰** (up)：视差 rate = 0.3 - 0.5，位置靠前
- **中景装饰** (article)：视差 rate = 0.7 - 0.9
- **近景装饰** (down)：视差 rate = 0.9 - 1.0，在角色前

#### 实现步骤
1. 创建装饰管理器脚本
2. 在 game_manager.gd 中加载和初始化
3. 根据 lorien.cfg 的位置和顺序配置

---

### 阶段 4：修复角色多层渲染 (后期)

**目标**：让角色看起来像原版一样完整

#### 角色图层 (swordman 示例)
- skin
- pants
- coat
- weapon
- hair
- face
- eyes
- ... (共约 20 个图层)

#### 实现思路
- 每个图层使用独立的 AnimatedSprite2D
- 保持相同的原点和缩放
- 播放相同的动画，但可能有微小的时间差或偏移

---

## 当前文件需要修改/创建

| 文件路径 | 操作 | 状态 |
|---------|-----|------|
| `scenes/player.tscn` | 添加缩放和原点 | 📝 待处理 |
| `scenes/enemy.tscn` | 添加缩放和原点 | 📝 待处理 |
| `scripts/game_manager.gd` | 集成装饰系统 | 📝 待处理 |
| `scenes/parallax_background.tscn` | 可能需要增强 | 📝 待处理 |
| `scripts/decorator_manager.gd` | 实现装饰逻辑 | ✅ 已准备 |
| `doc/041_SPRITE_FIX.md` | 精灵修复文档 | 📝 待创建 |
| `doc/042_DECORATION_IMPLEMENTATION.md` | 装饰实现文档 | 📝 待创建 |
| `doc/043_FINAL_VERIFICATION.md` | 最终验证文档 | 📝 待创建 |

---

## 下一步行动

1. **创建修复文档**
2. **修复基础精灵配置**
3. **实现装饰系统**
4. **测试并验证**

---

## 相关参考文件

- `config/map/making/lorien.cfg` - 地图配置
- `config/asset/sprite/map/lorien/` - 装饰配置
- `config/actor/avatar/duelist/swordman.cfg` - 角色配置
- `source/map/background.lua` - 背景渲染
- `source/graphics/drawable/sprite.lua` - 精灵渲染
