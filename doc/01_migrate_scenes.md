# Phase 15: Scene and Player Migration

## Overview

创建游戏场景和玩家控制器，从基础框架过渡到可运行的游戏原型。

## Goals

1. 创建主菜单场景
2. 创建游戏场景
3. 实现玩家控制器
4. 设置相机跟随

## Implementation

### Scene Files

- `scenes/main_menu.tscn` - 主菜单
- `scenes/game.tscn` - 游戏主场景
- `scenes/player.tscn` - 玩家角色

### Scripts

- `player_controller.gd` - 玩家控制器
- `game_manager.gd` - 游戏管理器

## Godot Scene Setup

### Player Scene Structure

```
Player (CharacterBody2D)
├── Sprite2D (visual)
├── CollisionShape2D (collision)
├── AnimationPlayer (animations)
└── GPUParticles2D (effects)
```

### Game Scene Structure

```
Node2D (Main)
├── World (Node2D)
│   ├── TileMap (floor)
│   ├── Player (CharacterBody2D)
│   └── Enemies (Node2D)
├── Camera2D (follow player)
├── UI (CanvasLayer)
│   └── HUD (Control)
└── GameManager
```

## Next Steps

- Phase 16: Enemy and AI Implementation
- Phase 17: Game Flow (menu, pause, game over)