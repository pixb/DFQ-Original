# Phase 20: Sound Effects

## Overview

添加音效系统，支持跳跃、攻击、受伤等音效。

## Implementation

### audio_system.gd

```gdscript
class_name AudioSystem extends Node

var sfx_players = {}

func play_sound(name: String, volume: float = 1.0):
    # 播放音效
func stop_sound(name: String):
    # 停止音效
```

### Sound Types

| Sound | Trigger |
|-------|----------|
| jump | 玩家跳跃时 |
| attack | 玩家攻击时 |
| dash | 冲刺时 |
| hurt | 受伤时 |
| enemy_death | 敌人死亡时 |

### Integration

- PlayerController 调用音效
- Enemy 调用音效

## Next Steps

- Phase 21: Main Menu
- Add BGM music