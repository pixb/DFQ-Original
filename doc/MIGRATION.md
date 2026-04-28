# DFQ 项目迁移计划 (LÖVE → Godot)

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

### 阶段 22: 修复画面显示 ⭐

**目标**: 修复游戏画面显示问题

**任务**:
1. 添加相机系统
2. 设置背景色
3. 修复碰撞形状
4. 解决Sprite2D纹理问题

**预计工作量**: 1-2 小时

**当前状态**: ✅ 完成

**相关文档**: 022_fix_display.md

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

**已完成阶段**: 1-14, 22

**原则**:
1. 从底层到上层
2. 先基础后复杂
3. 每阶段可独立测试
4. 按依赖关系排序

**当前进度**: 约 **75%** 完成（阶段1-14核心功能 + 阶段22画面修复）

**可运行性**:
- ✅ 玩家控制（移动、跳跃、冲刺、攻击）
- ✅ 敌人AI（巡逻、追击）
- ✅ 地板碰撞
- ✅ 游戏流程（暂停、游戏结束、胜利）
- ⏳ 真实精灵图（暂用调试矩形）
- ⏳ 血条UI更新（连接中）
- ⏳ 背景音乐
- ⏳ 攻击伤害系统

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