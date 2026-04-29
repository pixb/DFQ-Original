# Phase 35: 天气特效系统

## 概述

本阶段实现Godot天气特效系统，包括雨、雪、雾等天气效果。

## 天气类型

| 天气 | 描述 | 效果 |
|------|------|------|
| **雨** | 下落的雨滴效果 | 粒子系统 |
| **雪** | 飘落的雪花效果 | 粒子系统 |
| **雾** | 环境雾气效果 | 后处理 |
| **雷暴** | 闪电和雷声 | 视觉+音频 |
| **晴天** | 明亮的光照 | 光照调整 |

## 实现方案

### 1. 天气管理器

```gdscript
# scripts/weather_manager.gd
class_name WeatherManager
extends Node

enum WeatherType { SUNNY, RAIN, SNOW, FOG, STORM }

var current_weather: WeatherType = WeatherType.SUNNY
var weather_effects: Dictionary = {}

func _ready():
    initialize_effects()
    print("WeatherManager initialized")

func initialize_effects():
    # 创建各种天气效果节点
    weather_effects["rain"] = RainEffect.new()
    weather_effects["snow"] = SnowEffect.new()
    weather_effects["fog"] = FogEffect.new()
    weather_effects["storm"] = StormEffect.new()
    
    for name in weather_effects:
        weather_effects[name].name = name + "_effect"
        weather_effects[name].visible = false
        add_child(weather_effects[name])

func set_weather(weather_type: WeatherType):
    # 停止当前天气
    stop_current_weather()
    
    # 设置新天气
    current_weather = weather_type
    
    # 启动新天气效果
    match weather_type:
        WeatherType.RAIN:
            weather_effects["rain"].start()
        WeatherType.SNOW:
            weather_effects["snow"].start()
        WeatherType.FOG:
            weather_effects["fog"].start()
        WeatherType.STORM:
            weather_effects["storm"].start()
            weather_effects["rain"].start()
    
    print("Weather changed to: ", weather_type)

func stop_current_weather():
    for name in weather_effects:
        weather_effects[name].stop()

func toggle_rain():
    if current_weather == WeatherType.RAIN:
        set_weather(WeatherType.SUNNY)
    else:
        set_weather(WeatherType.RAIN)
```

### 2. 雨滴效果类

```gdscript
# scripts/weather/rain_effect.gd
class_name RainEffect
extends Node2D

var particles: Array = []
var is_active: bool = false
var rain_count: int = 100

func _ready():
    z_index = 50

func start():
    is_active = true
    visible = true
    spawn_rain()
    print("Rain effect started")

func stop():
    is_active = false
    visible = false
    clear_rain()
    print("Rain effect stopped")

func spawn_rain():
    for i in range(rain_count):
        var drop = Line2D.new()
        drop.width = 1
        drop.color = Color(0.6, 0.8, 1.0, 0.7)
        drop.points = [Vector2(0, 0), Vector2(0, 20)]
        drop.position = Vector2(rand_range(0, 1280), rand_range(-100, 720))
        add_child(drop)
        particles.append(drop)

func clear_rain():
    for drop in particles:
        if is_instance_valid(drop):
            drop.queue_free()
    particles.clear()

func _process(delta):
    if not is_active:
        return
    
    for drop in particles:
        if is_instance_valid(drop):
            drop.position += Vector2(2, 150) * delta * 60
            
            # 重置超出屏幕的雨滴
            if drop.position.y > 720:
                drop.position.y = -20
                drop.position.x = rand_range(0, 1280)
```

### 3. 雪花效果类

```gdscript
# scripts/weather/snow_effect.gd
class_name SnowEffect
extends Node2D

var particles: Array = []
var is_active: bool = false
var snow_count: int = 80

func _ready():
    z_index = 50

func start():
    is_active = true
    visible = true
    spawn_snow()
    print("Snow effect started")

func stop():
    is_active = false
    visible = false
    clear_snow()
    print("Snow effect stopped")

func spawn_snow():
    for i in range(snow_count):
        var flake = Sprite2D.new()
        flake.texture = load("res://asset/weather/snowflake.png")
        flake.scale = Vector2(rand_range(0.5, 1.5), rand_range(0.5, 1.5))
        flake.position = Vector2(rand_range(0, 1280), rand_range(-100, 720))
        flake.rotation = rand_range(0, PI * 2)
        add_child(flake)
        particles.append({"sprite": flake, "speed": rand_range(20, 60), "drift": rand_range(-20, 20)})

func clear_snow():
    for p in particles:
        if is_instance_valid(p["sprite"]):
            p["sprite"].queue_free()
    particles.clear()

func _process(delta):
    if not is_active:
        return
    
    for p in particles:
        if is_instance_valid(p["sprite"]):
            p["sprite"].position.y += p["speed"] * delta
            p["sprite"].position.x += sin(OS.get_ticks_msec() / 500 + p["drift"]) * delta * 30
            p["sprite"].rotation += delta
            
            if p["sprite"].position.y > 720:
                p["sprite"].position.y = -20
                p["sprite"].position.x = rand_range(0, 1280)
```

### 4. 雾气效果类

```gdscript
# scripts/weather/fog_effect.gd
class_name FogEffect
extends CanvasLayer

var fog_rect: ColorRect
var is_active: bool = false

func _ready():
    layer = 100
    
    fog_rect = ColorRect.new()
    fog_rect.anchors_preset = 15
    fog_rect.anchor_right = 1.0
    fog_rect.anchor_bottom = 1.0
    fog_rect.color = Color(0.8, 0.85, 0.9, 0.0)
    add_child(fog_rect)

func start():
    is_active = true
    visible = true
    animate_fog(true)
    print("Fog effect started")

func stop():
    is_active = false
    animate_fog(false)
    print("Fog effect stopped")

func animate_fog(fade_in: bool):
    var target_alpha = 0.3 if fade_in else 0.0
    
    var tween = Tween.new()
    add_child(tween)
    
    tween.interpolate_property(fog_rect, "color:a", fog_rect.color.a, target_alpha, 1.0)
    tween.start()
    
    await tween.finished
    tween.queue_free()
```

### 5. 雷暴效果类

```gdscript
# scripts/weather/storm_effect.gd
class_name StormEffect
extends Node

var lightning_flash: ColorRect
var is_active: bool = false

func _ready():
    lightning_flash = ColorRect.new()
    lightning_flash.anchors_preset = 15
    lightning_flash.anchor_right = 1.0
    lightning_flash.anchor_bottom = 1.0
    lightning_flash.color = Color.WHITE
    lightning_flash.visible = false
    add_child(lightning_flash)

func start():
    is_active = true
    schedule_lightning()
    print("Storm effect started")

func stop():
    is_active = false
    print("Storm effect stopped")

func schedule_lightning():
    if not is_active:
        return
    
    await get_tree().create_timer(rand_range(2.0, 8.0)).timeout
    
    if is_active:
        strike_lightning()
        schedule_lightning()

func strike_lightning():
    lightning_flash.visible = true
    
    var tween = Tween.new()
    add_child(tween)
    
    tween.interpolate_property(lightning_flash, "color:a", 1.0, 0.0, 0.1)
    tween.start()
    
    await tween.finished
    lightning_flash.visible = false
    tween.queue_free()
    
    # 播放雷声
    var audio_manager = get_node_or_null("/root/AudioManager")
    if audio_manager:
        audio_manager.play_sfx("res://asset/sfx/thunder.mp3")
```

## 集成到游戏管理器

```gdscript
# game_manager.gd
var weather_manager: WeatherManager

func _ready():
    # ... 其他初始化 ...
    setup_weather_manager()

func setup_weather_manager():
    weather_manager = WeatherManager.new()
    weather_manager.name = "WeatherManager"
    add_child(weather_manager)
    print("WeatherManager setup")

func toggle_weather():
    var weathers = [
        WeatherManager.WeatherType.SUNNY,
        WeatherManager.WeatherType.RAIN,
        WeatherManager.WeatherType.SNOW,
        WeatherManager.WeatherType.FOG,
        WeatherManager.WeatherType.STORM
    ]
    
    var current_index = weathers.find(weather_manager.current_weather)
    var next_index = (current_index + 1) % weathers.size()
    weather_manager.set_weather(weathers[next_index])
```

## 输入绑定

在 `project.godot` 中添加：

```ini
[input]
weather_toggle={
"deadzone": 0.5,
"events": [ Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":0,"alt":false,"shift":false,"control":false,"meta":false,"command":false,"pressed":false,"keycode":69,"physical_keycode":0,"unicode":0,"echo":false,"script":null) ]
}
```

## 测试验证

### 测试用例

| 测试项 | 操作 | 预期结果 |
|--------|------|----------|
| 切换天气 | 按E键 | 天气循环切换 |
| 雨滴效果 | 切换到雨天 | 雨滴下落 |
| 雪花效果 | 切换到雪天 | 雪花飘落 |
| 雾气效果 | 切换到雾天 | 屏幕变朦胧 |
| 雷暴效果 | 切换到雷暴 | 闪电+雷声 |

## 性能优化

### 1. 粒子数量控制
根据游戏性能动态调整粒子数量：

```gdscript
func update_particle_count(target_fps: float = 60.0):
    var current_fps = Engine.get_frames_per_second()
    var scale = current_fps / target_fps
    
    rain_count = int(100 * scale)
    snow_count = int(80 * scale)
```

### 2. 使用Particle2D节点
对于复杂效果，使用Godot内置的Particle2D节点。

### 3. 遮挡剔除
只在相机可见范围内生成粒子。

## 后续改进

- ✅ Phase 36: 完整地图系统
- ✅ Phase 37: 游戏优化与发布

## 相关文档

- weather_manager_gd.md - 天气管理器代码模板
- rain_effect_gd.md - 雨滴效果代码模板
- snow_effect_gd.md - 雪花效果代码模板
- fog_effect_gd.md - 雾气效果代码模板
- storm_effect_gd.md - 雷暴效果代码模板
