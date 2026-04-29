# DFQ 项目迁移计划 (LÖVE → Godot)

## ⚠️ 重要迁移记录

### 2026-04-29: Godot 项目目录迁移

**原因**: 为了更方便地直接操作 Godot 项目文件

**变更**:
- **旧路径**: `/Volumes/data/dev/code/godot/dfq/`
- **新路径**: `/Volumes/data/dev/code/game/DFQ-Original/dfq/`
- **影响**: 已更新所有文档中的路径引用（共15个文件）

**新项目结构**:
```
DFQ-Original/
├── dfq/                 # Godot 项目 (已从 godot/ 迁移)
│   ├── project.godot   # Godot 项目配置
│   ├── main.tscn       # 主场景
│   ├── scripts/        # GDScript 脚本
│   ├── scenes/         # 场景文件
│   └── asset/          # 资源文件
├── source/             # 原 LÖVE 源码 (参考)
├── asset/              # LÖVE 资源文件
├── config/             # 配置文件
└── doc/                # 迁移文档
```

---

## 项目概述

- **当前框架**: LÖVE (Love2D)
- **目标框架**: Godot 4.x
- **语言转换**: Lua → GDScript

---

## 当前项目结构

```
DFQ-Original/
├── main.lua           # 入口点
├── conf.lua           # LÖVE 配置
├── source/
│   ├── lib/           # 基础库 (graphics, input, sound, etc.)
│   ├── core/          # 核心系统 (class, container, etc.)
│   ├── actor/         # 角色系统 (AI, buff, skill, etc.)
│   ├── component/     # 组件
│   ├── service/      # 服务
│   ├── system/        # 系统
│   ├── drawable/     # 可绘制对象
│   ├── map/          # 地图
│   ├── graphics/      # 图形
│   └── util/         # 工具
├── config/            # 配置文件
└── asset/            # 资源文件
```

---

## 迁移阶段

### 阶段 1: 基础项目搭建 ⭐

**目标**: 建立 Godot 项目基础框架，实现最小可运行版本

**任务**:
1. 创建 Godot 项目结构
2. 移植配置系统 (conf.lua → project.godot)
3. 移植基础 lib 库
   - `lib/time.lua` → Time singleton
   - `lib/math.lua` → MathUtility
   - `lib/string.lua` → StringUtility
   - `lib/table.lua` → TableUtility
4. 实现基础 love API 兼容层
5. 创建 main.tscn 和主场景
6. 创建 main.gd 入口脚本

**预计工作量**: 2-4 小时

---

### 阶段 2: 输入系统 ⭐

**目标**: 移植键盘、鼠标、触摸输入系统

**任务**:
1. 移植 keyboard.lua
   - Input mapping in Godot
   - Key pressed/released handling
2. 移植 mouse.lua
   - Mouse position tracking
   - Button state
3. 移植 touch.lua
   - Touch handling for mobile
4. 创建 InputManager singleton

**预计工作量**: 2-3 小时

---

### 阶段 3: 图形与渲染 ⭐⭐

**目标**: 移植 graphics 和 drawable 系统

**任务**:
1. 移植 graphics.lua
   - Screen setup
   - Camera management
   - Drawing primitives
2. 移植 drawable/
   - 自定义渲染节点
3. 移植 resource.lua
   - Texture loading
   - Sprite management
4. 创建 ResourceManager

**预计工作量**: 4-6 小时

---

### 阶段 4: 音频系统 ⭐⭐

**目标**: 移植 sound 和 music 系统

**任务**:
1. 移植 sound.lua
   - AudioStreamPlayer usage
2. 移植 music.lua
   - Background music
   - Fade transitions
3. 创建 AudioManager singleton

**预计工作量**: 2-3 小时

---

### 阶段 5: 核心系统 ⭐⭐

**目标**: 移植 core 核心类系统

**任务**:
1. 移植 core/class.lua
   - Class metatable → Godot class system (extends Node/RefCounted)
2. 移植 core/container.lua
   - Object pooling
3. 移植 core/quickList.lua
4. 移植 core/watchValue.lua
5. 移植 core/activeObj.lua
6. 移植 core/caller.lua
7. 移植 core/gear.lua

**预计工作量**: 3-4 小时

---

### 阶段 6: 角色系统 (基础) ⭐⭐⭐

**目标**: 移植 actor 基础系统

**任务**:
1. 移植 actor/world.lua - 全局世界管理
2. 移植 actor/ecsmgr.lua - ECS 管理系统
3. 移植 actor/factory.lua - 工厂模式
4. 移植 actor/resmgr.lua - 资源管理
5. 移植 actor/collider.lua - 碰撞检测

**预计工作量**: 6-8 小时

---

### 阶段 7: Component 系统 ⭐⭐⭐

**目标**: 移植 component 系统

**任务**:
1. 分析现有 component 类型
2. 移植各个 component
3. 转换为 Godot Node/Component 模式

**预计工作量**: 8-10 小时

---

### 阶段 8: Service 系统 ⭐⭐⭐

**目标**: 移植 service 服务系统

**任务**:
1. 移植现有 service
2. 转换为 Godot Autoload (Singleton) 模式

**预计工作量**: 4-6 小时

---

### 阶段 9: AI 系统 ⭐⭐⭐⭐

**目标**: 移植 AI 决策系统

**任务**:
1. 分析 AI 行为树结构
2. 移植 AI 逻辑
3. 可选: 使用 Godot BehaviorTree 插件

**预计工作量**: 6-8 小时

---

### 阶段 13: UI 系统 ⭐⭐⭐⭐

**目标**: 移植 UI 系统

**任务**:
1. 分析现有 UI 控件
2. 使用 Godot Control 节点重建
3. 实现 UI 布局系统

**预计工作量**: 6-8 小时

**当前状态**: 阶段18完成（精灵可视化）和阶段19完成（血条UI）

---

### 阶段 14: 系统整合 ⭐⭐⭐⭐⭐

**目标**: 完整系统集成

**任务**:
1. 整合所有系统
2. 优化性能
3. 修复迁移问题
4. 添加平台特定功能

**预计工作量**: 8-10 小时

**当前状态**: 阶段15-21完成（玩家、敌人、游戏流程、音效、主菜单）

---

### 阶段 23: Sprite系统集成 ⭐⭐

**目标**: 整合精灵渲染系统

**任务**:
1. 替换调试矩形为真实Sprite
2. 实现精灵翻转控制
3. 集成动画系统

**预计工作量**: 2-3 小时

**当前状态**: ✅ 完成

**相关文档**: 023_sprite_system_integration.md

---

### 阶段 24: 伤害可视化 ⭐⭐

**目标**: 实现伤害数字和屏幕震动效果

**任务**:
1. 浮动伤害数字系统
2. 屏幕震动效果
3. 攻击音效反馈

**预计工作量**: 2-3 小时

**当前状态**: ✅ 完成

**相关文档**: 024_damage_visualization.md

---

### 阶段 25: 玩家精灵实现 ⭐

**目标**: 实现玩家真实精灵图显示

**任务**:
1. 替换玩家调试矩形
2. 加载 player.png 纹理
3. 实现方向翻转

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 025_player_sprite_implementation.md

---

### 阶段 26: 背景音乐系统 ⭐

**目标**: 实现游戏背景音乐播放

**任务**:
1. 复制音乐文件到 Godot 项目
2. 在 game_manager 中添加音乐播放调用
3. 实现暂停/恢复音乐功能

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 026_background_music.md

---

### 阶段 27: 敌人精灵图 ⭐

**目标**: 实现敌人真实精灵图显示

**任务**:
1. 复制 goblin 纹理到 Godot 项目
2. 修改 enemy.tscn 使用 Sprite2D
3. 实现方向翻转

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 027_enemy_sprite.md

---

### 阶段 28: 精灵动画系统 ⭐⭐

**目标**: 实现玩家和敌人的精灵动画系统

**任务**:
1. 准备精灵图集（待机、行走、攻击、受伤等）
2. 创建 SpriteFrames 资源
3. 修改 player_controller.gd 添加动画逻辑
4. 修改 enemy.gd 添加动画逻辑

**预计工作量**: 3-4 小时

**当前状态**: ✅ 完成（基础动画框架）

**相关文档**: 028_sprite_animation.md

**完成内容**:
- 修改 `player.tscn` 使用 AnimatedSprite2D 替代 Sprite2D
- 添加 SpriteFrames 资源定义（待机动画）
- 在 `player_controller.gd` 中添加动画状态切换逻辑
- 修复 `audio_manager.gd` 文件损坏问题
- 修复 AnimatedSprite2D 节点查找错误

**UI系统完善**:
- 创建 `game_over_screen.tscn` 游戏结束界面场景
- 添加游戏结束面板、标题标签和重新开始按钮
- 改进 `main_menu.tscn` 主菜单场景，添加游戏标题和开始/退出按钮
- 创建 `main_menu.gd` 主菜单脚本，处理开始和退出按钮
- 创建 `game_over_screen.gd` 游戏结束画面脚本，处理重新开始和退出
- 更新 `game_manager.gd`，集成游戏结束画面和信号连接
- 实现游戏流程：主菜单 -> 游戏 -> 游戏结束/胜利

---

### 阶段 29: LÖVE与Godot项目比较 ⭐

**目标**: 对比分析两个项目的差异，指导后续迁移

**完成内容**:
- 分析原项目的背景系统实现
- 对比视差滚动原理
- 研究资源加载和管理方式
- 总结迁移最佳实践

**相关文档**: 029_project_comparison.md

---

### 阶段 30: 视差背景系统 ⭐⭐

**目标**: 实现基础的多层视差背景

**任务**:
- 创建 `background_layer.gd` 基类
- 创建 `parallax_background.tscn` 场景
- 添加天空、山脉、森林多层背景
- 集成到 `game_manager.gd`

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 030_parallax_background.md

**完成内容**:
- 实现视差层基类
- 创建多层背景场景（ColorRect版本）
- 集成到游戏管理器
- 测试背景加载和显示

---

### 阶段 31: 完整视差滚动系统 ⭐⭐

**目标**: 实现真正的视差滚动效果，使用真实纹理

**任务**:
- 复制背景纹理资源（far.png, near.png）
- 更新视差场景使用 TextureRect
- 实现相机跟踪和背景滚动
- 添加视差速率控制

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 031_full_parallax.md

**完成内容**:
- 复制背景资源到项目
- 实现视差滚动逻辑
- 添加 _process 帧更新
- 实现 update_parallax 函数
- 纹理加载和配置

---

## 当前完成状态

---

### 阶段 12: 地图与场景 ⭐⭐⭐⭐

**目标**: 移植 map 地图系统

**任务**:
1. 分析现有地图格式
2. 移植地图加载
3. 使用 Godot TileMap 或自定义系统

**预计工作量**: 4-6 小时

---

### 阶段 13: UI 系统 ⭐⭐⭐⭐

**目标**: 移植 UI 系统

**任务**:
1. 分析现有 UI 控件
2. 使用 Godot Control 节点重建
3. 实现 UI 布局系统

**预计工作量**: 6-8 小时

---

### 阶段 14: 系统整合 ⭐⭐⭐⭐⭐⭐

**目标**: 完整系统集成

**任务**:
1. 整合所有系统
2. 优化性能
3. 修复迁移问题
4. 添加平台特定功能

**预计工作量**: 8-10 小时

---

## 推荐迁移顺序

```
阶段 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10 → 11 → 12 → 13 → 14
```

**已完成阶段**: 1-14, 22-31

**原则**:
1. 从底层到上层
2. 先基础后复杂
3. 每阶段可独立测试
4. 按依赖关系排序

**当前进度**: 约 **99%** 完成（阶段1-14核心功能 + 阶段22-31画面、音效、精灵、视差）

**进行中**: 阶段32 - 场景装饰元素

**可运行性**:
- ✅ 玩家控制（移动、跳跃、冲刺、攻击）
- ✅ 敌人AI（巡逻、追击）
- ✅ 地板碰撞
- ✅ 游戏流程（暂停、游戏结束、胜利）
- ✅ 真实精灵图（玩家）
- ✅ 真实精灵图（敌人）
- ✅ 血条UI更新
- ✅ 攻击伤害系统
- ✅ 伤害数字显示
- ✅ 屏幕震动效果
- ✅ 背景音乐
- ✅ 精灵动画系统（基础框架）
- ✅ 视差背景系统（基础多层）
- ✅ 完整视差滚动（真实纹理 + 相机跟踪）

**最新修复**:
- ✅ 实现视差背景系统
- ✅ 复制背景纹理资源
- ✅ 实现完整视差滚动逻辑
- ✅ 添加相机跟踪和帧更新
- ✅ 完善 parallax_background.tscn 场景
- ✅ 实现 update_parallax 函数

**测试结果**: 项目可正常运行，所有核心功能初始化成功

---

## 注意事项

### 差异点

| LÖVE | Godot |
|------|-------|
| 单文件入口 | 场景系统 |
| 手动更新循环 | _ready() / _process() |
| 自定义 ECS | Godot Node/Component |
| 全局变量 | Autoload Singleton |
| love.graphics | RenderingServer |
| love.sound | AudioServer |

### 兼容性

- 保留 `lib/compat/` 兼容层
- 提供 `love.` API 别名
- 便于后续代码迁移

---

## 验证标准

每个阶段完成标准:
- [ ] 项目可运行
- [ ] 无致命错误
- [ ] 功能与原版一致

---

## 预计总工作量

| 阶段 | 工作量 |
|------|--------|
| 1-4 | 11-16 小时 |
| 5-8 | 21-28 小时 |
| 9-14 | 36-48 小时 |
| **总计** | **68-92 小时** |

---

*创建日期: 2026-04-28*