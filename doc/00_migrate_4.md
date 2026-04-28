# 阶段 4: 音频系统 ✅

**目标**: 移植 sound 和 music 系统

---

## 任务清单

- [x] 4.1 AudioManager
- [x] 4.2 SFX 播放
- [x] 4.3 Music 播放 (淡入淡出)
- [x] 4.4 音量控制

---

## 已创建

```
scripts/audio_manager.gd    # 音频管理 (Autoload)
```

## AudioManager 功能

| 方法 | 功能 |
|------|------|
| `play_sfx()` | 播放音效 |
| `play_music()` | 播放音乐 (带淡入淡出) |
| `pause_music()` | 暂停音乐 |
| `stop_music()` | 停止音乐 |
| `set_sfx_volume()` | 设置音量 |
| `set_music_volume()` | 设置音量 |

## 验证结果

```
- AudioManager initialized: ✓
- Autoload: ✓
```

## 音频映射

| LÖVE | Godot |
|------|-------|
| love.sound | AudioStreamPlayer |
| love.music | AudioStreamPlayer + Tween |

---

*阶段 4 完成*