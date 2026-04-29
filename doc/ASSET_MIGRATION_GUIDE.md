# LÖVE2D 素材迁移到 Godot 指南

## 概述

LÖVE2D 和 Godot 都支持标准的素材格式，因此**所有 LÖVE2D 素材都可以直接迁移到 Godot 使用**。

## 素材格式兼容性

| 类型 | LÖVE2D支持 | Godot支持 | 兼容性 |
|------|-----------|----------|--------|
| **图像** | PNG, JPG, BMP, GIF | PNG, JPG, BMP, WEBP | ✅ 完全兼容 |
| **音频** | OGG, WAV, MP3 | OGG, WAV, MP3 | ✅ 完全兼容 |
| **字体** | TTF, OTF | TTF, OTF | ✅ 完全兼容 |

## 素材迁移步骤

### 步骤 1: 复制素材文件

从原项目复制到 Godot 项目：

```bash
# 复制纹理素材
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/textures/

# 复制音乐素材
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/sound/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/music/

# 复制特效素材
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/actor/effect/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/effects/
```

### 步骤 2: 在 Godot 中加载素材

#### 方法 1: 通过 Sprite2D 加载单张图片

```gdscript
# 在脚本中动态加载
var sprite: Sprite2D = Sprite2D.new()
sprite.texture = load("res://asset/textures/player.png")
add_child(sprite)
```

#### 方法 2: 通过 AnimatedSprite2D 加载动画帧

在 Godot 编辑器中：
1. 创建 AnimatedSprite2D 节点
2. 在 Inspector 中创建 SpriteFrames 资源
3. 导入所有动画帧图片

#### 方法 3: 使用 TextureRect 加载背景

```gdscript
var bg: TextureRect = TextureRect.new()
bg.texture = load("res://asset/textures/background.png")
bg.anchors_preset = 15  # 全屏
add_child(bg)
```

### 步骤 3: 实现视差背景（使用真实素材）

```gdscript
# 创建视差层
func create_parallax_layer(texture_path: String, parallax_rate: float, y_position: float):
    var layer = ParallaxLayer.new()
    layer.parallax_scale = Vector2(parallax_rate, 1.0)
    
    var sprite = Sprite2D.new()
    sprite.texture = load(texture_path)
    sprite.position = Vector2(0, y_position)
    sprite.scale = Vector2(2, 2)  # 根据需要缩放
    
    layer.add_child(sprite)
    $ParallaxBackground.add_child(layer)
```

### 步骤 4: 设置精灵动画

```gdscript
# player_controller.gd
func _ready():
    var animated_sprite = $AnimatedSprite2D
    animated_sprite.frames = load("res://asset/player_frames.tres")
    animated_sprite.play("idle")
```

## 关键注意事项

### 1. 素材路径格式
- **LÖVE2D**: `asset/image/player.png`
- **Godot**: `res://asset/textures/player.png`

### 2. 像素完美渲染
在 `project.godot` 中设置：
```ini
[display]
window/size/width=1280
window/size/height=720
window/stretch/mode="viewport"
window/stretch/aspect="keep"
```

### 3. 精灵锚点设置
```gdscript
sprite.anchor_point = Vector2(0.5, 0.5)  # 居中对齐
sprite.pivot_offset = Vector2(0, sprite.texture.get_height()/2)  # 底部对齐
```

### 4. 动画帧速率
```gdscript
animated_sprite.speed_scale = 0.5  # 调整动画速度
```

## 完整示例：加载敌人精灵

```gdscript
# enemy.gd
class_name Enemy extends CharacterBody2D

var sprite: AnimatedSprite2D
var is_walking: bool = false

func _ready():
    sprite = $AnimatedSprite2D
    
    # 加载精灵帧资源
    var frames = SpriteFrames.new()
    
    # 添加行走帧
    for i in range(4):
        frames.add_frame("walk", load("res://asset/textures/enemy/walk_" + str(i) + ".png"))
    
    # 添加攻击帧
    for i in range(3):
        frames.add_frame("attack", load("res://asset/textures/enemy/attack_" + str(i) + ".png"))
    
    sprite.frames = frames
    sprite.play("walk")

func update_animation():
    if is_walking:
        sprite.play("walk")
    else:
        sprite.play("idle")
```

## 性能优化建议

### 1. 使用纹理图集
将小图片合并成大图集，减少 draw call。

### 2. 压缩纹理
在 Godot 中设置纹理压缩格式。

### 3. 异步加载
```gdscript
await get_tree().create_timer(0.1).timeout
var texture = load("res://asset/textures/big_image.png")
```

## 素材目录结构建议

```
godot/dfq/
├── asset/
│   ├── textures/          # 静态纹理
│   │   ├── player/
│   │   ├── enemy/
│   │   ├── map/
│   │   └── ui/
│   ├── effects/           # 特效素材
│   │   ├── hit/
│   │   ├── buff/
│   │   └── death/
│   ├── music/             # 背景音乐
│   └── fonts/            # 字体文件
```

## 总结

| 项目 | 说明 |
|------|------|
| **兼容性** | ✅ 所有格式完全兼容 |
| **迁移难度** | 低 - 只需复制文件 |
| **注意事项** | 路径格式、锚点设置、渲染模式 |
| **优化建议** | 使用纹理图集、异步加载 |

---

*创建日期: 2026-04-29*
