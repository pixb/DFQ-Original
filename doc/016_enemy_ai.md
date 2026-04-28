# Phase 16: Enemy and AI Implementation

## Overview

敌人和AI系统实现，包括敌人基础类、状态机和追逐逻辑。

## LÖVE Reference

- `source/actor/component/ai.lua`
- `source/actor/system/duelist/init.lua`

## Implementation

### enemy.gd

敌人 AI 基类 (CharacterBody2D):

**状态机:**
- IDLE: 待机状态
- PATROL: 巡逻
- CHASE: 追逐玩家
- ATTACK: 攻击范围
- HIT: 受伤
- DEAD: 死亡

**属性:**
- health/max_health: 生命值
- speed: 移动速度
- patrol_range: 巡逻范围
- chase_range: 追逐范围
- attack_range: 攻击范围

**方法:**
- ai_idle(): 待机AI
- ai_patrol(): 巡逻AI  
- ai_chase(): 追逐AI
- ai_attack(): 攻击AI
- take_damage(amount): 受到伤害
- die(): 死亡
- set_target(): 设置目标

### 敌人行为

1. **巡逻**: 在起始点附近来回移动
2. **发现目标**: 距离小于 chase_range 时进入追逐
3. **追逐**: 加速向目标移动
4. **攻击**: 进入攻击范围后停止并攻击
5. **返回**: 目标超出范围后返回巡逻

### test_results

- Player spawned at (640.0, 360.0)
- Spawned 3 enemies

## Next Steps

- Phase 17: Game Flow (menu, pause, game over)
- Add collision with floor
- Add attack damage to player
- Add health bar UI