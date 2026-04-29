# 场景装饰元素

## 概述

在完善的视差背景基础上，添加原项目中的场景装饰元素，包括拱门、树木、石头等，使游戏场景更丰富。

## 装饰资源

原项目中的装饰资源位于 `asset/image/map/lorien/`：

### 树木
- `tree/0.png` - `tree/10.png` - 各种树木精灵

### 石头
- `stone/0.png` - `stone/5.png` - 各种石头精灵

### 花
- `flower/0.png` - `flower/4.png` - 各种花精灵

### 草
- `grass/0.png` - `grass/3.png` - 各种草精灵

### 石柱
- `stonePillar/0.png` - `stonePillar/3.png` - 石柱精灵

### 路径
- `trail/0.png` - `trail/2.png` - 路径精灵

### 地砖
- `tile/0.png` - `tile/3.png` - 地砖精灵

## 实现方案

### 1. 装饰元素基类 (`scripts/decoration.gd`)

```gdscript
class_name Decoration
extends Node2D

var texture: Texture2D
var parallax_rate: float = 0.5

func _init(texture_path: String, rate: float = 0.5):
    texture = load(texture_path)
    parallax_rate = rate
```

### 2. 装饰层场景 (`scenes/decoration_layer.tscn`)

```gdscript
[gd_scene format=3]

[node name="DecorationLayer" type="Node2D"]
```

### 3. 装饰管理器

在 `game_manager.gd` 中添加装饰管理：

```gdscript
var decorations: Array = []

func setup_decorations():
    var dec_layer = Node2D.new()
    dec_layer.name = "Decorations"
    add_child(dec_layer)
    
    # 添加树木
    add_decoration(dec_layer, "res://asset/map/lorien/tree/0.png", Vector2(200, 400), 0.4)
    add_decoration(dec_layer, "res://asset/map/lorien/tree/1.png", Vector2(500, 380), 0.4)
    
    # 添加石头
    add_decoration(dec_layer, "res://asset/map/lorien/stone/0.png", Vector2(800, 450), 0.6)
    
    print("Decorations setup complete")

func add_decoration(parent: Node2D, texture_path: String, position: Vector2, rate: float):
    var sprite = Sprite2D.new()
    var texture = load(texture_path)
    
    if texture:
        sprite.texture = texture
        sprite.position = position
        sprite.set_meta("parallax_rate", rate)
        parent.add_child(sprite)
        decorations.append(sprite)

func update_decorations(camera_pos: Vector2):
    for dec in decorations:
        if is_instance_valid(dec):
            var rate = dec.get_meta("parallax_rate", 0.5)
            dec.position = Vector2(
                dec.get_meta("original_x", dec.position.x) - camera_pos.x * rate,
                dec.position.y
            )
```

### 4. 在 _process 中集成

```gdscript
func _process(delta):
    if current_state == GameState.PLAYING and player:
        update_parallax()
        update_decorations(camera.position)
```

## 实现步骤

### 步骤 1: 复制装饰资源

```bash
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/tree
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/stone
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/flower
mkdir -p /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/grass

cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/tree/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/tree/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/stone/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/stone/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/flower/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/flower/
cp -r /Volumes/data/dev/code/game/DFQ-Original/asset/image/map/lorien/grass/* /Volumes/data/dev/code/game/DFQ-Original/dfq/asset/map/lorien/grass/
```

### 步骤 2: 创建装饰层场景

创建 `scenes/decoration_layer.tscn` 作为装饰元素的容器。

### 步骤 3: 实现装饰管理器

在 `game_manager.gd` 中添加装饰加载和管理逻辑。

### 步骤 4: 添加视差滚动支持

让装饰元素也参与视差滚动，创造更丰富的深度感。

## 装饰布局建议

参考原项目的布局：

```
场景布局 (从左到右):
├── 远景: 山脉
├── 中景: 大树
├── 近景: 石头、花草
└── 前景: 玩家和敌人
```

## 性能优化

- 使用对象池管理装饰元素
- 只加载和渲染可见区域的装饰
- 使用纹理图集减少 draw call
- 合理设置装饰的视差速率

## 后续优化

- 添加装饰元素动画
- 实现装饰元素交互（如可破坏的石头）
- 添加更多装饰类型
- 实现装饰随机生成系统

## 测试

运行项目后应该看到：
1. 树木装饰在背景适当位置
2. 石头装饰在地面
3. 花草装饰点缀场景
4. 所有装饰随相机移动产生视差效果
