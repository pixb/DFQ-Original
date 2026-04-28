# Phase 11: Skill System Migration

## Overview

迁移技能系统，包括技能基础类、技能数据、技能组件和技能服务。

## LÖVE Reference Files

- `source/actor/skill/base.lua` - Skill base class
- `source/actor/skill/beaten.lua` - Beaten skill
- `source/actor/skill/buff.lua` - Buff skill
- `source/actor/component/skills.lua` - Skills component
- `source/actor/system/skills.lua` - Skills system
- `source/actor/service/skill.lua` - Skill service

## Godot Implementation

### skill_base.gd

基础技能类，包含:
- timer: Timer - 技能计时器
- entity: Entity - 所属实体
- judge_ai: AiComponent - AI判断
- data: SkillData - 技能数据
- key: String - 技能键
- mp, time, state, cool_down, order, attack_values, dura_max, dura, is_combo, hp_rate, is_ultimate

方法:
- update(dt) - 更新技能状态
- can_use() - 检查技能是否可用
- cond() - 检查技能条件
- is_active() - 技能是否激活
- use() - 使用技能
- cool_down_timer(force) - 冷却
- reset() - 重置
- in_cool_down() - 是否冷却中
- get_now_time/set_now_time - 获取/设置当前时间
- get_process() - 获取进度

### skill_data.gd

技能数据类，从资源配置加载:
- path, origin, state, time, mp, order, dura
- now_time, in_cool_down, beaten_time
- hp_rate, is_ultimate
- attack_values, buff, ai_data

### timer.gd

Timer 工具类:
- elapsed, target, is_running
- enter(time) - 开始计时
- exit() - 结束计时
- update(dt) - 更新
- get_process() - 获取进度

### skills_component.gd

技能组件:
- container: Container - 技能容器
- caller: Caller - 事件调用器
- data: Dictionary - 技能数据

### skills_service.gd

技能服务:
- _keys: ["normalAttack", "skill1", "skill2", "skill3", "skill4", "skill5", "skill6"]
- set_skill(entity, key, data) - 设置技能
- add(entity, data) - 添加技能
- get_skill_with_path(skills, path) - 根据路径获取
- get_skill_with_origin(skills, origin) - 根据origin获取

### skills_system.gd

技能系统:
- add_entity/remove_entity - 实体管理
- on_enter(entity) - 进入回调
- update(dt) - 更新所有技能

## Integration

- Services singleton added skill service
- ResourceManager handles skill data loading

## Testing

- Skill container test: CoreUtils.Container works
- Phase 11 Complete