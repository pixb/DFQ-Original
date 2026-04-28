# DFQ-Original

DFQ-Original is a basic version of Dungeon & Fighter·Quest (DFQ), it's a coterie game of [DNF](http://dnf.qq.com).

## 运行方式

### 环境要求

- [LÖVE 0.10.2](https://bitbucket.org/rude/love/downloads/) 或更高版本

### 安装 LÖVE

```bash
# macOS
brew install love

# Linux
sudo apt install love

# Windows
# 从 https://love2d.org 下载安装
```

### 运行游戏

```bash
# 直接运行项目目录
love /Volumes/data/dev/code/game/DFQ-Original

# 或打包成 .love 文件运行
cd /Volumes/data/dev/code/game/DFQ-Original
zip -r game.love *.lua source/ asset/ config/ conf.lua
love game.love
```

## Code Standards

DFQ has own-style code standards, see below:

* Private variable: prefixed with `_` => `_a`
* Function & Class: prefixed with capital letter => `Test()`
* Module (A table but not class): all capital letter => `MAP`

And DFQ use a code hinting plugin named [EmmyLua](https://github.com/EmmyLua/VSCode-EmmyLua), so you can see some comments such like `---@xxx`.

## 项目结构

```
DFQ-Original/
├── main.lua          # 游戏入口
├── conf.lua          # LÖVE 配置文件
├── source/           # 源代码
│   ├── lib/          # 基础库（time, mouse, keyboard, graphics, sound 等）
│   ├── director/     # 导演（场景管理）
│   └── ...
├── asset/            # 资源文件（图片、音效等）
└── config/           # 配置文件
```

## About

You can learn more in [my blog](https://musoucrow.github.io).