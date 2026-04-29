# LÖVE2D资源迁移到Godot计划

## 概述

本计划详细说明如何将LÖVE2D项目的图片资源迁移到Godot中使用。

## 资源目录分析

### 1. 地图资源 (map/lorien/)

| 资源类型 | 文件数 | 用途 |
|---------|-------|------|
| flower/ | 5个 | 花朵装饰 |
| grass/ | 4个 | 草地装饰 |
| stone/ | 6个 | 石头装饰 |
| stonePillar/ | 4个 | 石柱装饰 |
| tile/ | 4个 | 地图瓦片 |
| trail/ | 3个 | 小路纹理 |
| tree/ | 11个 | 树木装饰 |
| far.png | 1个 | 远景背景 |
| near.png | 1个 | 近景背景 |

### 2. 角色资源 (actor/)

| 角色类型 | 部位 | 文件数 |
|---------|------|-------|
| goblin | skin/hair/pants/coat/weapon | 每个31帧 |
| lugaru | skin | 37帧 |
| swordman | skin/face/hair/coat | 每4-5帧 |
| tau | skin/pants/hat/weapon/armlet | 每个32帧 |

### 3. 特效资源 (effect/)

| 特效类型 | 子类型 | 文件数 |
|---------|--------|-------|
| hitting | fire/dark/water/light/magic | 9-13帧/类型 |
| buff | bleed/fear/freeze/haste/slow | 5-24帧/类型 |
| death | normal | 12帧 |
| summon | back/front/bottom | 20帧/类型 |
| mark | large/small | 10-15帧 |

## 迁移步骤

### Phase A: 创建Godot资源目录结构

```bash
# 创建目录结构
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/actor/duelist
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/hitting
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/buff
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/death
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/summon
```

### Phase B: 复制地图资源

```bash
# 复制地图装饰
cp /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/*.png \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien/

# 复制地图子目录
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/flower \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/grass \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/stone \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/tree \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/map/lorien/
```

### Phase C: 复制角色资源

```bash
# 复制角色纹理
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/duelist \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/actor/
```

### Phase D: 复制特效资源

```bash
# 复制攻击特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/hitting \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/

# 复制Buff特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/buff \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/

# 复制死亡特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/death \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/

# 复制召唤特效
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/summon \
   /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/effects/
```

## 在Godot中使用资源

### 1. 更新视差背景使用真实纹理

```gdscript
# scripts/parallax_layer.gd
func _ready():
    # 使用真实纹理替换ColorRect
    var bg_sprite = Sprite2D.new()
    bg_sprite.texture = load("res://asset/textures/map/lorien/far.png")
    bg_sprite.scale = Vector2(2, 2)
    add_child(bg_sprite)
```

### 2. 更新装饰系统使用真实纹理

```gdscript
# scripts/decorator_manager.gd
func add_tree(position: Vector2, tree_type: int = 0):
    var sprite = Sprite2D.new()
    sprite.texture = load("res://asset/textures/map/lorien/tree/" + str(tree_type) + ".png")
    sprite.position = position
    add_child(sprite)
```

### 3. 更新角色精灵使用真实纹理

```gdscript
# scripts/player_controller.gd
func _ready():
    var animated_sprite = $AnimatedSprite2D
    var frames = SpriteFrames.new()
    
    # 加载角色动画帧
    for i in range(4):
        var texture = load("res://asset/textures/actor/duelist/swordman/skin/default/" + str(i) + ".png")
        frames.add_frame("idle", texture)
    
    animated_sprite.frames = frames
    animated_sprite.play("idle")
```

### 4. 更新特效系统使用真实纹理

```gdscript
# scripts/effect_manager.gd
func play_fire_effect(position: Vector2):
    var effect = AnimatedSprite2D.new()
    var frames = SpriteFrames.new()
    
    for i in range(13):
        var texture = load("res://asset/textures/effects/hitting/fire/" + str(i) + ".png")
        frames.add_frame("default", texture)
    
    effect.frames = frames
    effect.position = position
    effect.play("default")
    add_child(effect)
```

## 资源路径映射表

| LÖVE2D路径 | Godot路径 |
|------------|-----------|
| asset/image/map/lorien/ | res://asset/textures/map/lorien/ |
| asset/image/actor/duelist/ | res://asset/textures/actor/duelist/ |
| asset/image/actor/effect/hitting/ | res://asset/textures/effects/hitting/ |
| asset/image/actor/effect/buff/ | res://asset/textures/effects/buff/ |
| asset/image/actor/effect/death/ | res://asset/textures/effects/death/ |
| asset/image/actor/effect/summon/ | res://asset/textures/effects/summon/ |

## 迁移任务清单

### ✅ Phase A: 目录结构
- [ ] 创建地图纹理目录
- [ ] 创建角色纹理目录
- [ ] 创建特效纹理目录

### ✅ Phase B: 地图资源迁移
- [ ] 复制花朵装饰
- [ ] 复制草地装饰
- [ ] 复制石头装饰
- [ ] 复制树木装饰
- [ ] 复制视差背景

### ✅ Phase C: 角色资源迁移
- [ ] 复制swordman纹理
- [ ] 复制goblin纹理
- [ ] 复制tau纹理
- [ ] 复制lugaru纹理

### ✅ Phase D: 特效资源迁移
- [ ] 复制攻击特效
- [ ] 复制Buff特效
- [ ] 复制死亡特效
- [ ] 复制召唤特效

### ✅ Phase E: 代码更新
- [ ] 更新视差背景使用真实纹理
- [ ] 更新装饰系统使用真实纹理
- [ ] 更新角色精灵使用真实纹理
- [ ] 更新特效系统使用真实纹理

### ✅ Phase F: 测试验证
- [ ] 测试地图渲染
- [ ] 测试角色显示
- [ ] 测试特效播放

## 预期效果

迁移完成后，Godot项目将拥有：

1. **精美的视差背景** - 使用原项目的far.png和near.png
2. **丰富的地图装饰** - 树木、石头、花草等
3. **完整的角色动画** - 所有角色的动画帧
4. **绚丽的特效系统** - 攻击、Buff、死亡、召唤特效

## 注意事项

1. **纹理格式** - 所有PNG格式完全兼容
2. **路径转换** - LÖVE2D相对路径→Godot res://路径
3. **动画帧顺序** - 保持原项目的帧顺序
4. **性能优化** - 考虑使用纹理图集

## 后续优化

1. **纹理图集** - 将小纹理合并成图集
2. **异步加载** - 大纹理使用异步加载
3. **内存管理** - 实现资源缓存机制

---

*创建日期: 2026-04-29*
