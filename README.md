# Roguelike Shooter — Godot 4.6 肉鸽射击游戏 Skill

[![Godot](https://img.shields.io/badge/Godot-4.6-478cbf?logo=godot-engine&logoColor=white)](https://godotengine.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

一个用于 **Godot 4.6** 俯视角 2D 肉鸽 (roguelike) 射击游戏的开发模板 / Skill。提供完整的游戏架构设计、核心系统实现指南和编码规范。

---

## 这是什么？

这是一个 **Skill**（`SKILL.md`），当你使用Claude Code 环境时，可以通过斜杠命令加载它，快速获得一个肉鸽射击游戏从零到完整的开发指导。

技能内容涵盖了游戏的整体架构、每个子系统的设计思路、代码模板，以及推荐的开发路线。

## 核心系统

| 系统 | 说明 |
|------|------|
| 房间探索 | 随机生成战斗/宝箱/Boss 房间，TileMap + 走廊连接 |
| 玩家 | WASD 移动、鼠标瞄准射击、右键闪避、无敌帧 |
| 武器 | Resource 驱动，支持散射/穿透/元素/击退 |
| 敌人 | 追踪/射击/自爆/冲锋/Boss 五种类型 |
| 强化升级 | 过关三选一，稀有度分级，属性叠加 Build |
| 经济掉落 | 金币/经验/道具，击杀掉落 + 商店购买 |
| 永久死亡 | 每局独立，全局解锁，多角色选择 |

## 快速开始

### 方式一：作为Skill 使用

1. 将 `roguelike-shooter` 文件夹放入你的skills 目录
2. 在 Claude Code 中输入 `/roguelike-shooter` 加载技能
3. 按技能指导逐步开发

### 方式二：作为开发参考

直接阅读 `SKILL.md` 获取：
- 完整的项目目录结构
- 每个系统的代码模板和设计思路
- GDScript 编码规范
- 分阶段开发路线图

## 技术栈

- **引擎**: Godot 4.6 (Forward Plus)
- **语言**: GDScript（强制类型标注）
- **分辨率**: 480×360 (canvas_items 拉伸)
- **物理**: Godot 默认 2D 物理
- **碰撞层**: 按层位运算管理

## 编码风格

本项目遵循约定的 GDScript 编码规范：

```gdscript
extends ClassName

const MAX_HP := 10

@export var speed: float = 120.0
@onready var sprite: AnimatedSprite2D = $Sprite

var _is_dead := false

func _ready() -> void:
    add_to_group("player")
    signal_name.connect(_on_signal)
```

完整规范见 `SKILL.md`。

## 开发路线

```
Phase 1 ─ 基础框架   →  Phase 2 ─ 战斗系统   →  Phase 3 ─ 肉鸽循环   →  Phase 4 ─ 打磨
```

## 许可证

[MIT](LICENSE)
