# Godot SpriteFrames配置详细步骤

## 方法1: 使用Godot编辑器界面操作

### 完整步骤

#### 1. 打开SpriteFrames编辑器

1. 在Godot编辑器中，找到刚创建的 `asset/frames/player_sprites.tres`
2. 双击打开它

#### 2. 添加"idle"动画

1. 在右上角的 **Animations** 面板中，点击 **新建动画** 按钮（"+"号）
2. 在弹出的对话框中，输入动画名称：**idle**
3. 点击确认

#### 3. 添加"idle"的动画帧

1. 在左侧文件系统面板中，导航到：
   `asset/textures/actor/duelist/swordman/skin/default/`

2. 按住 **Ctrl** 键（Windows）或 **Cmd** 键（Mac），选择以下文件：
   - `0.png`
   - `1.png`
   - `2.png`
   - `3.png`

3. 将这4个文件**拖拽**到右侧的 **Animation Frames** 列表区域
4. 释放鼠标，这些文件将作为动画帧添加

#### 4. 设置"idle"动画属性

1. 在动画列表中选择 "idle"
2. 在下方的动画属性面板中：
   - 设置 **Speed** (速度) 为 5.0
   - 勾选 **Loop** (循环播放) 复选框

#### 5. 添加"walk"动画

1. 再次点击 **新建动画** 按钮（"+"号）
2. 输入动画名称：**walk**
3. 点击确认

#### 6. 添加"walk"的动画帧

1. 同样选择那4个文件（0.png到3.png）
2. 拖拽到walk动画的帧列表区域

#### 7. 设置"walk"动画属性

1. 选择 "walk" 动画
2. 设置：
   - **Speed** (速度) 为 8.0
   - 勾选 **Loop** (循环播放)

#### 8. 保存

按 **Ctrl+S** 或 **Cmd+S** 保存资源文件

---

## 方法2: 直接创建完整的TRES文件

如果您想直接创建，我为您准备了完整的文件内容。请在Godot项目目录中创建：

### 文件位置
`/Volumes/data/dev/code/game/DFQ-Original/dfq/asset/frames/player_sprites.tres`

### 完整内容

```ini
[gd_resource type="SpriteFrames" load_steps=5 format=3]

[ext_resource type="Texture2D" path="res://asset/textures/actor/duelist/swordman/skin/default/0.png" id="1"]
[ext_resource type="Texture2D" path="res://asset/textures/actor/duelist/swordman/skin/default/1.png" id="2"]
[ext_resource type="Texture2D" path="res://asset/textures/actor/duelist/swordman/skin/default/2.png" id="3"]
[ext_resource type="Texture2D" path="res://asset/textures/actor/duelist/swordman/skin/default/3.png" id="4"]

[sub_resource type="SpriteFrames"]
animations = [
{
  "name": "idle",
  "speed": 5.0,
  "loop": true,
  "frames": [
    ExtResource("1"),
    ExtResource("2"),
    ExtResource("3"),
    ExtResource("4")
  ]
},
{
  "name": "walk",
  "speed": 8.0,
  "loop": true,
  "frames": [
    ExtResource("1"),
    ExtResource("2"),
    ExtResource("3"),
    ExtResource("4")
  ]
}
]
```

### 创建步骤

1. 在Finder中打开 `/Volumes/data/dev/code/game/DFQ-Original/dfq/asset/frames/`
2. 右键点击 `player_sprites.tres`
3. 选择"打开方式" → "文本编辑"
4. 将上面的内容粘贴进去
5. 保存文件
6. 在Godot编辑器中重新打开项目或刷新项目

---

## 方法3: 在Godot中使用内置编辑器的界面提示

### 界面区域说明

| 区域 | 说明 |
|------|------|
| 顶部 | 动画列表（已创建的动画） |
| 左侧 | 文件浏览器（找到PNG文件） |
| 右侧 | 动画帧列表（拖放这里） |
| 底部 | 当前动画的属性 |

### 快捷操作

| 快捷键 | 功能 |
|--------|------|
| Ctrl+N | 新建动画 |
| Delete | 删除选中帧 |
| Ctrl+S | 保存 |
| 鼠标滚轮 | 缩放时间线 |

---

## 验证配置

配置完成后，您可以：

### 1. 预览动画

在SpriteFrames编辑器中，选择一个动画，点击 **播放** 按钮（▶）

### 2. 在Player场景中使用

1. 打开 `scenes/player.tscn`
2. 选择 `AnimatedSprite2D` 节点
3. 在检查器中找到 **Frames** 属性
4. 点击右侧的箭头按钮，选择 `asset/frames/player_sprites.tres`
5. 在检查器中设置：
   - **Animation** = idle
   - **Playing** = true
6. 在场景视图中您应该能看到角色动画在播放！

### 3. 运行测试

按 **F5** 运行项目，检查：
- 角色是否显示为正确的精灵
- 移动时是否播放walk动画
- 停止时是否播放idle动画

---

## 常见问题

### Q: 拖放图片到帧列表没有反应？
A: 确保您先选择了正确的动画（在动画列表中点击动画名称）

### Q: 动画播放速度太快/太慢？
A: 在动画属性中调整Speed值，数值越大越快

### Q: 只显示一张图片？
A: 确保Animation属性已设置，Playing已勾选

### Q: 动画顺序不对？
A: 可以在帧列表中拖拽调整帧的顺序

---

## 后续步骤

配置完成后，请参考：
[GODOT_CONFIGURATION_GUIDE.md](GODOT_CONFIGURATION_GUIDE.md)
[CONFIGURATION_CHECKLIST.md](CONFIGURATION_CHECKLIST.md)
