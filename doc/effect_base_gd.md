# EffectBase 特效基础类

## 文件位置

scripts/effect_base.gd

## 完整代码

```gdscript
class_name EffectBase
extends Node2D

signal finished

var sprite: Sprite2D
var animation_player: AnimationPlayer
var frames: SpriteFrames
var is_playing: bool = false
var duration: float = 1.0
var elapsed_time: float = 0.0
var target_node: Node = null
var follow_target: bool = false
var auto_remove: bool = true
var animation_speed: float = 1.0

func _ready():
    sprite = $Sprite2D
    animation_player = $AnimationPlayer
    
    if animation_player:
        animation_player.animation_finished.connect(_on_animation_finished)
    
    print("EffectBase initialized")

func play(anim_name: String = "default"):
    is_playing = true
    elapsed_time = 0.0
    
    if animation_player and animation_player.has_animation(anim_name):
        animation_player.play(anim_name, -1.0, animation_speed)
    elif frames and sprite:
        sprite.play(anim_name)

func stop():
    is_playing = false
    
    if animation_player:
        animation_player.stop()
    elif frames and sprite:
        sprite.stop()
    
    if auto_remove:
        finished.emit()

func _process(delta):
    if not is_playing:
        return
    
    elapsed_time += delta
    
    # 跟随目标
    if follow_target and target_node:
        if is_instance_valid(target_node):
            position = target_node.position
    
    # 超时自动结束
    if elapsed_time >= duration:
        stop()

func _on_animation_finished(anim_name: String):
    if auto_remove:
        finished.emit()

func set_follow(target: Node):
    target_node = target
    follow_target = true

func set_duration(time: float):
    duration = time

func set_speed(speed: float):
    animation_speed = speed
    if animation_player:
        animation_player.playback_speed = speed
```
