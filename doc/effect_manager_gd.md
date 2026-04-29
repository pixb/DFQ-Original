# EffectManager 特效管理器

## 文件位置

scripts/effect_manager.gd

## 完整代码

```gdscript
class_name EffectManager
extends Node

var effect_scenes: Dictionary = {}
var effect_instances: Array = []
var active_effects: Array = []

func _ready():
    load_effect_scenes()
    print("EffectManager initialized")

func load_effect_scenes():
    # 加载攻击特效
    effect_scenes["hit_fire"] = load("res://scenes/effects/hit_fire.tscn")
    effect_scenes["hit_dark"] = load("res://scenes/effects/hit_dark.tscn")
    effect_scenes["hit_water"] = load("res://scenes/effects/hit_water.tscn")
    effect_scenes["hit_light"] = load("res://scenes/effects/hit_light.tscn")
    effect_scenes["hit_small_slash1"] = load("res://scenes/effects/hit_small_slash1.tscn")
    effect_scenes["hit_large_slash1"] = load("res://scenes/effects/hit_large_slash1.tscn")
    effect_scenes["hit_guard"] = load("res://scenes/effects/hit_guard.tscn")
    effect_scenes["hit_blood"] = load("res://scenes/effects/hit_blood.tscn")
    effect_scenes["hit_explosion"] = load("res://scenes/effects/hit_explosion.tscn")

    # 加载Buff特效
    effect_scenes["buff_bleed"] = load("res://scenes/effects/buff_bleed.tscn")
    effect_scenes["buff_fear"] = load("res://scenes/effects/buff_fear.tscn")
    effect_scenes["buff_freeze"] = load("res://scenes/effects/buff_freeze.tscn")
    effect_scenes["buff_haste"] = load("res://scenes/effects/buff_haste.tscn")
    effect_scenes["buff_slow"] = load("res://scenes/effects/buff_slow.tscn")
    effect_scenes["buff_heal"] = load("res://scenes/effects/buff_heal.tscn")

    # 加载死亡特效
    effect_scenes["death_normal"] = load("res://scenes/effects/death_normal.tscn")
    effect_scenes["death_flash"] = load("res://scenes/effects/death_flash.tscn")

    # 加载召唤特效
    effect_scenes["summon_back"] = load("res://scenes/effects/summon_back.tscn")
    effect_scenes["summon_front"] = load("res://scenes/effects/summon_front.tscn")
    effect_scenes["summon_bottom"] = load("res://scenes/effects/summon_bottom.tscn")

    print("Effect scenes loaded: ", effect_scenes.size())

func play_effect(effect_name: String, position: Vector2, parent: Node = null):
    if not effect_scenes.has(effect_name):
        print("Effect not found: ", effect_name)
        return null
    
    var effect_scene: PackedScene = effect_scenes[effect_name]
    if not effect_scene:
        return null
        
    var effect_instance: Node2D = effect_scene.instantiate()
    
    effect_instance.position = position
    
    if parent:
        parent.add_child(effect_instance)
    else:
        get_tree().current_scene.add_child(effect_instance)
    
    effect_instances.append(effect_instance)
    active_effects.append(effect_instance)
    
    # 自动移除
    effect_instance.finished.connect(func():
        if is_instance_valid(effect_instance):
            effect_instance.queue_free()
            effect_instances.erase(effect_instance)
            active_effects.erase(effect_instance))
    
    return effect_instance

func play_hit_effect(hit_type: String, position: Vector2, target_node: Node = null):
    var effect_name = "hit_" + hit_type
    var effect = play_effect(effect_name, position, target_node)
    return effect

func play_buff_effect(buff_type: String, target: Node):
    var effect_name = "buff_" + buff_type
    var effect = play_effect(effect_name, target.position, target)
    if effect:
        # 跟随目标
        effect.target_node = target
    return effect

func play_death_effect(position: Vector2):
    var effect = play_effect("death_normal", position)
    return effect

func play_flash_death_effect(position: Vector2):
    var effect = play_effect("death_flash", position)
    return effect

func play_summon_effect(position: Vector2, layer: String = "front"):
    var effect_name = "summon_" + layer
    var effect = play_effect(effect_name, position)
    return effect

func play_weather_effect(weather_type: String):
    var effect_name = "weather_" + weather_type
    var effect = play_effect(effect_name, Vector2.ZERO)
    return effect

func clear_all_effects():
    for effect in effect_instances:
        if is_instance_valid(effect):
            effect.queue_free()
    effect_instances.clear()
    active_effects.clear()
