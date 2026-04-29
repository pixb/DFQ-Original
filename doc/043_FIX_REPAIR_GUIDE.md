# 修复指南

## 当前状态

✅ decorations.tscn 装饰场景已创建并复制到 Godot 项目
✅ 相机跟随和敌人位置修复方案已准备

## 需要手动操作

### 1. 手动复制 decorations.tscn 到 Godot 项目

文件已创建在：`/Volumes/data/dev/code/game/DFQ-Original/dfq/scenes/decorations.tscn`

### 2. 修改 game_manager.gd

编辑 `/Volumes/data/dev/code/game/DFQ-Original/dfq/scripts/game_manager.gd`

#### 添加变量（在 var game_over_screen: CanvasLayer 后面）

```gdscript
var camera: Camera2D = null
var camera_follow_speed: float = 0.1
var camera_offset: Vector2 = Vector2(640, 360)
```

#### 修改 setup_camera()

```gdscript
func setup_camera():
    camera = Camera2D.new()
    camera.position = Vector2(640, 360)
    camera.current = true
    camera.smoothing_enabled = true
    camera.smoothing_speed = 5.0
    add_child(camera)
    print("Camera setup with smoothing")
```

#### 添加 _process() 函数（在 setup_camera() 后面）

```gdscript
func _process(delta: float):
    if camera and player and is_instance_valid(player):
        var target_pos = player.global_position + camera_offset
        camera.global_position = camera.global_position.lerp(target_pos, camera_follow_speed)
```

#### 修改 setup_enemies() 中的敌人位置

```gdscript
enemy.position = Vector2(1200 + i * 300, 400)
```

### 3. 运行项目测试

在 Godot 编辑器中运行项目，验证：

- [ ] 相机是否跟随玩家移动
- [ ] 敌人是否在远离玩家的位置生成（1200+）
- [ ] 装饰场景是否正确加载（树、草、花、石头）

## 文件清单

| 文件 | 位置 | 状态 |
|------|------|------|
| decorations.tscn | `/Volumes/data/dev/code/game/DFQ-Original/dfq/scenes/` | ✅ 已创建 |
