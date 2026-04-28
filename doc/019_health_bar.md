# Phase 19: Health Bar UI

## Overview

创建血条UI显示玩家生命值。

## Implementation

### health_bar.tscn

- ProgressBar 节点
- max_value: 100
- value: 100

### game_manager.gd

更新添加血条显示逻辑：
- player_health: 玩家生命值
- max_player_health: 最大生命值

### 显示位置

- UI 层显示在 CanvasLayer

## Test Results

- Game started
- Floor created  
- Player spawned at (640.0, 360.0)
- 3 enemies spawned

## Next Steps

- Phase 20: Sound Effects
- Connect health bar to player damage