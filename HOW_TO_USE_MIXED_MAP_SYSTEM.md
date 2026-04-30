# 🎮 混合地图系统使用指南

## ✨ 什么是混合地图系统？

**混合地图系统** = **代码自动生成** + **Godot编辑器手动调整**

我们把地图分成了两部分：

1. **代码生成层**（自动）
   - 背景视差（Far、Near）
   - 地板地砖（Floor）
   - 基础装饰（由 `map_generator.gd` 生成）

2. **手动编辑层**（你手动调整）
   - `scenes/decorations.tscn` - 你的像素艺术画板！
   - 精确放置的树木、石头、花朵、大门...
   - 你在 Godot 编辑器里直接拖放即可！

---

## 🛠️ 3步开始像素级复刻你的地图！

### 第一步：打开项目，运行游戏看现状

1. 打开 Godot 4.6+，打开项目：`/Volumes/data/dev/code/game/DFQ-Original/dfq/`
2. 按 **F5** 或点击 ▶️ 运行游戏
3. 看看当前地图和你原始截图的差异

---

### 第二步：打开 `decorations.tscn` 开始手动调整

1. 在 Godot 的文件系统面板中找到：`scenes/decorations.tscn`
2. **双击打开**它 - 这是你的手动装饰画板！
3. 在这个场景中你可以：

#### ✨ 操作指南
- **移动现有元素**：选中节点，在 2D 编辑器中直接拖动，或调整属性面板中的 `Position`
- **添加新元素**：右键 → **继承场景**，或直接在现有节点上右键 → **复制**，然后改纹理和位置
- **调整 Z 轴顺序**：选中节点，调整 `z_index`（数值大的在上面）
- **改变纹理**：选择一个 Sprite2D，在属性面板点击 `Texture` 旁边的下拉，选择你要的图片

#### 📂 可用的纹理资源
在 `asset/textures/map/lorien/` 里有：

| 文件夹 | 内容 |
|--------|------|
| `flower/` | 各种花朵（0-4） |
| `grass/` | 基础草（0-3） |
| `largeGrass/` | 大型草（0-7） |
| `smallGrass/` | 小型草（0-7） |
| `tree/` | 各种树（0-10） |
| `stone/` | 石头（0-5） |
| `stonePillar/` | 石柱（0-3） |
| `trail/` | 路径（0-2） |
| `pathgate/` | 大门、灌木、路树等！ |

#### 🎯 如何调整到像素级准确？

对比你原来的截图：`/Volumes/data/dev/code/game/DFQ-Original/screenshot/entergame.png`

1. 在第二个窗口打开原始截图
2. 在 Godot 中逐元素调整：
   - 调整 `position`（精确到 1 像素）
   - 调整 `scale`（原始游戏用 1.3 左右）
   - 确保所有元素对齐！

---

### 第三步：保存，运行，看效果！

1. 在 `decorations.tscn` 中调整好后，**按 Ctrl+S 保存**
2. 回到主游戏场景（`game.tscn`），按 **F5** 运行
3. 现在你的手动装饰会 **自动加载** 到地图最上层！

---

## 🎯 工作流程建议（最佳实践）

### 方案 A：完全手动（适合像素完美复刻）

1. 禁用或简化 `map_generator.gd` 中的随机生成
2. 把所有装饰都放去 `decorations.tscn`
3. 你完全掌控每个像素！

### 方案 B：混合使用（推荐，我们现在就是这个）

1. 代码负责基础元素（背景、地板、基础的草和树）
2. 你负责关键元素（大门位置、重要的树木、你要的特定装饰）
3. 两者结合，效率最高！

### 方案 C：自动生成 + 微调

1. 用代码生成大部分元素
2. 只微调那些位置和原始截图差很多的元素到 `decorations.tscn`

---

## 💡 小技巧

1. **先看 debug 输出**：运行游戏时，调试窗口会打印：
   ```
   ✅ Loaded manual decorations: decorations.tscn
      Tip: Open decorations.tscn in Godot Editor to adjust manually!
   ```
   这样你知道手动层已经加载成功

2. **利用现有节点**：`decorations.tscn` 已经有一些预配置的装饰了，你可以直接调整它们的位置！

3. **多做备份**：
   - 调整满意了可以复制一份 `decorations.tscn.backup`
   - 或者直接用 Git 版本控制

4. **参考原始代码**：看看 `/Volumes/data/dev/code/game/DFQ-Original/source/map/assigner/granfloris.lua` - 它告诉你原始游戏是怎么放装饰的！

---

## 🚀 下一步

现在你有两个选择：

1. **直接用现在的系统** → 打开 `decorations.tscn`，开始调整吧！
2. **想要先看现状** → 先运行一下游戏，看看现在的样子，再决定怎么调整

你想让我帮你做什么？告诉我吧！
