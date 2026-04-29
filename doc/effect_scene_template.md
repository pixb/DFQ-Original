# 特效场景模板

## 基础特效场景 (scenes/effects/effect_base.tscn)

```gdscript
[gd_scene format=3]

[ext_resource type="Script" path="res://scripts/effect_base.gd" id="1_base"]

[node name="EffectBase" type="Node2D"]
script = ExtResource("1_base")

[node name="Sprite2D" type="Sprite2D" parent="."]

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]
```

## 火焰攻击特效 (scenes/effects/hit_fire.tscn)

```gdscript
[gd_scene format=3]

[ext_resource type="Script" path="res://scripts/effect_base.gd" id="1_base"]
[ext_resource type="Texture2D" uid="uid://xxxxx1" path="res://asset/effects/hitting/fire/0.png" id="2_fire0"]
[ext_resource type="Texture2D" uid="uid://xxxxx2" path="res://asset/effects/hitting/fire/1.png" id="2_fire1"]

[node name="HitFire" type="Node2D"]
script = ExtResource("1_base")

[node name="Sprite2D" type="AnimatedSprite2D" parent="."]
frames = SubResource("SpriteFrames_fire")
animation = "default"

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]

[sub_resource type="SpriteFrames" id="SpriteFrames_fire"]
animations = [
{
  "name": "default",
  "speed": 10.0,
  "loop": false,
  "frames": [
    ExtResource("2_fire0"),
    ExtResource("2_fire1")
  ]
}
]
```

## Buff流血特效 (scenes/effects/buff_bleed.tscn)

```gdscript
[gd_scene format=3]

[ext_resource type="Script" path="res://scripts/effect_base.gd" id="1_base"]
[ext_resource type="Texture2D" uid="uid://xxxxx3" path="res://asset/effects/buff/bleed/0.png" id="3_bleed0"]

[node name="BuffBleed" type="Node2D"]
script = ExtResource("1_base")

[node name="Sprite2D" type="AnimatedSprite2D" parent="."]
frames = SubResource("SpriteFrames_bleed")
animation = "default"

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]

[sub_resource type="SpriteFrames" id="SpriteFrames_bleed"]
animations = [
{
  "name": "default",
  "speed": 8.0,
  "loop": true,
  "frames": [
    ExtResource("3_bleed0")
  ]
}
]
```

## 死亡特效 (scenes/effects/death_normal.tscn)

```gdscript
[gd_scene format=3]

[ext_resource type="Script" path="res://scripts/effect_base.gd" id="1_base"]
[ext_resource type="Texture2D" uid="uid://xxxxx4" path="res://asset/effects/death/normal/0.png" id="4_death0"]

[node name="DeathNormal" type="Node2D"]
script = ExtResource("1_base")

[node name="Sprite2D" type="AnimatedSprite2D" parent="."]
frames = SubResource("SpriteFrames_death")
animation = "default"

[node name="AnimationPlayer" type="AnimationPlayer" parent="."]

[sub_resource type="SpriteFrames" id="SpriteFrames_death"]
animations = [
{
  "name": "default",
  "speed": 12.0,
  "loop": false,
  "frames": [
    ExtResource("4_death0")
  ]
}
]
```
