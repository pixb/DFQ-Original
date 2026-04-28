# DFQ - Dungeon Fighter Quest

## 项目信息

- **引擎**: Godot 4.6.x
- **语言**: GDScript
- **窗口**: 1280x720
- **类型**: 2D 动作游戏

## 项目位置

- **原 LÖVE 项目**: `/Volumes/data/dev/code/game/DFQ-Original/`
- **Godot 项目**: `/Volumes/data/dev/code/godot/dfq/`
├── project.godot        # Godot 项目配置
├── main.tscn           # 主场景
├── scripts/           # GDScript 脚本
├── addons/            # 插件 (MCP)
├── source/            # 原 LÖVE 源码 (参考)
├── asset/             # 资源文件
├── config/            # 配置文件
└── doc/              # 迁移文档
```

## 开发命令

### MCP 工具 (已配置 tugcantopaloglu/godot-mcp)

```bash
# 运行项目
godot_run_project

# 停止项目
godot_stop_project

# 获取输出
godot_get_debug_output

# 运行后等待帧
godot_game_wait frames=10

# 执行 GDScript
godot_game_eval code="..."

# 获取场景树
godot_game_get_scene_tree
```

### 常用操作

```bash
# 列出项目文件
godot_list_project_files extensions=[".gd", ".tscn"]

# 创建场景
godot_create_scene projectPath="." scenePath="scenes/test.tscn" rootNodeType="Node2D"

# 创建脚本
godot_create_script projectPath="." scriptPath="scripts/test.gd" extends="Node"

# 读取文件
godot_read_file filePath="scripts/main.gd" projectPath="."

# 写入文件
godot_write_file content="..." filePath="scripts/test.gd" projectPath="."
```

## 迁移状态

- [阶段 1] 基础项目搭建 ✅
- [阶段 2] 输入系统
- [阶段 3] 图形与渲染
- [阶段 4] 音频系统
- [阶段 5] 核心系统
- [阶段 6] 角色系统
- [阶段 7] Component 系统
- [阶段 8] Service 系统
- [阶段 9] AI 系统
- [阶段 10] Buff 系统
- [阶段 11] Skill 系统
- [阶段 12] 地图与场景
- [阶段 13] UI 系统
- [阶段 14] 系统整合

详细见 `doc/MIGRATION.md`

## 输入系统

LÖVE → Godot 映射:

| LÖVE | Godot |
|------|-------|
| love.keypressed | _input |
| love.mousepressed | InputEventMouseButton |
| love.touchpressed | InputEventScreenTouch |

## 渲染系统

| LÖVE | Godot |
|------|-------|
| love.graphics | RenderingServer |
| Sprite | Sprite2D |
| Text | Label |

## 音频系统

| LÖVE | Godot |
|------|-------|
| love.sound | AudioStreamPlayer |
| love.music | AudioStreamPlayer |

## 原型参考

原项目 LÖVE 代码位于 `source/` 目录，迁移时参考使用。

---

*最后更新: 2026-04-28*