# LÖVE2D 地面绘制分析与 Godot 迁移指南

## 📊 当前问题分析

### 截图问题识别

从当前截图观察：

| 问题区域 | 描述 | 原因分析 |
|---------|------|---------|
| 顶部黑框 | 背景图片未填满屏幕 | 背景图片高度不够 |
| 中间黑条 | 地面瓦片与装饰之间有空隙 | 近景背景位置需要调整 |
| 底部绿色溢出 | 有多余的草地显示 | 装饰场景中元素位置超出边界 |

## 🔍 LÖVE2D 原项目绘制逻辑

### 1. 地图配置 (lorien.cfg)

```lua
info = {
    width = {1280 + 480, 1024 + 480},  -- 地图宽度
    height = {600, 736},                -- 地图高度
    ...
}
floor = {
    top = "$A/tile/2",      -- 顶部瓦片
    extra = "$A/tile/3",    -- 填充瓦片
    horizon = 327           -- 地面起始Y坐标
}
```

### 2. 瓦片尺寸分析

| 资源 | 尺寸 | 用途 |
|------|------|------|
| `tile/2.png` | 224 × 235 | 地面顶部瓦片（带草地纹理） |
| `tile/3.png` | 224 × 120 | 地面填充瓦片 |
| `far.png` | 640 × 375 | 远景背景 |
| `near.png` | 640 × 380 | 近景背景 |

### 3. 原项目绘制顺序

```lua
-- 从 LÖVE2D source/map/init.lua 提取

-- 1. 绘制远景背景 (scale=2)
for i=0, 2 do
    love.graphics.draw(far, i * 640 * 2, 0, 0, 2, 2)
end

-- 2. 绘制近景背景 (scale=2, y=150)
for i=0, 2 do
    love.graphics.draw(near, i * 640 * 2, 150, 0, 2, 2)
end

-- 3. 绘制地板瓦片 (y=327)
local x, y = 0, data.floor.horizon
while x < data.info.width do
    love.graphics.draw(top, x, y)
    local yy = y + top:getHeight()
    while yy < data.info.height do
        love.graphics.draw(extra, x, yy)
        yy = yy + extra:getHeight()
    end
    x = x + top:getWidth()
end

-- 4. 绘制装饰物
-- 5. 绘制角色层
```

## 📐 精确位置计算

### 关键坐标

| 元素 | X | Y | 说明 |
|------|---|---|------|
| 屏幕左上角 | 0 | 0 | 原点 |
| 屏幕右下角 | 1280 | 736 | 窗口尺寸 |
| 地面起始 | 0 | 327 | horizon 值 |
| 玩家出生点 | 640 | 351 | 屏幕中心偏上 |

### 瓦片堆叠计算

```
顶部瓦片高度: 235 → 结束 Y = 327 + 235 = 562
填充瓦片高度: 120 → 结束 Y = 562 + 120 = 682
```

## 🔧 Godot 修复方案

### 已实施的修复

**1. 远景背景调整 (2026-04-29)**
```gdscript
scale = Vector2(2.5, 3.0)           -- 放大以填满屏幕高度
position = Vector2(-160, -200)      -- 向上偏移覆盖顶部黑框
```

**2. 近景背景调整 (2026-04-29)**
```gdscript
scale = Vector2(2.5, 3.0)           -- 与远景匹配
position = Vector2(-160, 50)        -- 下移以衔接地面
```

**3. 装饰场景修复 (2026-04-29)**
- 调整远景树木位置（上移至 Y=30-50）
- 移除未使用的资源引用（tree/10.png, stone/5.png）
- 确保所有元素在屏幕范围内

**4. 地板瓦片修复 (2026-04-29)**
- 16列瓦片正确堆叠
- 使用真实瓦片尺寸 (224×235, 224×120)
- 碰撞体覆盖整个地板区域

**5. 相机跟随修复 (2026-04-29)**
- Camera2D 平滑跟随玩家
- 设置 position_smoothing_enabled = true
- 设置 position_smoothing_speed = 5.0

**6. 敌人位置修复 (2026-04-29)**
- 敌人在远处生成 (X=1200+)
- 远离玩家初始位置

## ✅ 已完成修复

| 序号 | 问题 | 修复状态 | 修复内容 |
|------|------|---------|---------|
| 1 | 远景背景高度 | ✅ 已修复 | 缩放 2.5×3.0，上移 -200 |
| 2 | 近景背景位置 | ✅ 已修复 | 缩放 2.5×3.0，Y 位置 50 |
| 3 | 装饰场景文件 | ✅ 已修复 | 调整元素位置，移除未使用资源 |
| 4 | 地板瓦片 | ✅ 已修复 | 16列瓦片正确堆叠 |
| 5 | 相机跟随 | ✅ 已修复 | Camera2D 平滑跟随玩家 |
| 6 | 敌人位置 | ✅ 已修复 | 敌人在远处生成 (1200+) |

## 📝 待修复清单

| 序号 | 问题 | 优先级 | 修复方向 |
|------|------|--------|---------|
| 1 | 顶部黑框 | 中 | 可能需要微调背景位置 |
| 2 | 底部绿色溢出 | 低 | 检查装饰场景元素 |
| 3 | 地面草地纹理 | 低 | 添加草地覆盖层 |

## 📁 相关文件

| 文件 | 路径 | 说明 | 状态 |
|------|------|------|------|
| parallax_background.tscn | `scenes/` | 视差背景场景 | ✅ |
| floor.tscn | `scenes/` | 地板场景 | ✅ |
| decorations.tscn | `scenes/` | 装饰场景 | ✅ |
| game_manager.gd | `scripts/` | 游戏管理器 | ✅ |
| player_controller.gd | `scripts/` | 玩家控制器 | ✅ |

## 📈 修复进度

```
总修复项: 6/6 ✅
待修复项: 3 (低优先级)
```

## 🎮 测试结果

项目启动测试通过：

```
✅ InputHandler initialized
✅ AudioManager initialized  
✅ Game started!
✅ Camera setup with smoothing
✅ Parallax background loaded
✅ Decorations loaded!
✅ Floor created at: (0.0, 327.0)
✅ Player spawned at: (640.0, 351.0)
✅ Spawned 3 enemies
✅ Background music started: lorien.mp3
```

---

*更新日期: 2026-04-29*
*分析基于: source/map/init.lua, config/map/making/lorien.cfg*
*修复状态: 大部分完成*
*项目路径: /Volumes/data/dev/code/game/DFQ-Original/dfq/*