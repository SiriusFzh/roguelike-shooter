---
name: roguelike-shooter
description: "Godot 4.6 肉鸽射击游戏开发。俯视角 2D 射击 + 房间探索 + 强化升级 + 永久死亡。遵循 F:\\CLAUDE.md 规范。"
description_zh: "Godot 4.6 肉鸽射击游戏开发"
description_en: "Roguelike shooter game development with Godot 4.6"
version: 0.1.0
---

# 肉鸽射击游戏开发 Skill

## 概述

基于 Godot 4.6，开发俯视角 2D 肉鸽(roguelike)射击游戏。遵循 `F:\CLAUDE.md` 中定义的 GDScript 编码规范。

## 核心系统

### 1. 房间探索系统

```
Game.tscn (根节点: Node)
├── Player         # 玩家
├── Camera2D       # 跟随相机
├── RoomManager    # 房间生成与管理
├── HUD            # 血条、金币、武器信息
├── GameUI         # 暂停、升级选择等覆盖UI
└── Background     # 背景/装饰层
```

**房间生成流程：**
```
RoomManager
├── _ready() → 生成起始房间
├── 玩家进入门区域 → 加载相邻房间
├── 房间表：
│   ├── 起始房间 (无怪)
│   ├── 战斗房间 (敌人波次)
│   ├── 宝箱房间 (奖励)
│   └── Boss房间
```

**TileMap 碰撞约定：**
- 图层 0: 地面 (walkable)
- 图层 1: 墙壁/障碍 (collision)
- 图层 2: 装饰 (无碰撞)

### 2. 玩家系统

```gdscript
extends CharacterBody2D
class_name Player

@export var move_speed: float = 120.0
@export var max_hp: int = 6
@export var bullet_speed: float = 320.0
@export var weapon: Resource  # WeaponResource
@export var fire_rate: float = 0.2
@export var pickup_range: float = 30.0

var hp: int: set = _set_hp
var current_room: Room
var _is_rolling := false
var _invul_timer := 0.0
```

**无敌帧实现：**
```gdscript
func take_damage(amount: int) -> void:
    if _invul_timer > 0.0:
        return
    hp -= amount
    _invul_timer = 0.5
    _flash_effect()
    if hp <= 0:
        _die()
```

### 3. 武器系统 (Resource 驱动)

```gdscript
extends Resource
class_name WeaponResource

@export var weapon_name: String
@export var damage: int = 1
@export var fire_rate: float = 0.2
@export var bullet_scene: PackedScene
@export var bullet_count: int = 1
@export var spread_angle: float = 0.0
@export var pierce_count: int = 0
@export var knockback: float = 0.0
@export var elemental: int = 0  # ElementalType enum
```

**枚举：**
```gdscript
enum WeaponType { SINGLE, BURST, SPREAD, SHOTGUN, LASER, BOOMERANG }
enum ElementalType { NONE, FIRE, ICE, POISON, LIGHTNING }
```

### 4. 敌人系统

```gdscript
extends CharacterBody2D
class_name Enemy

@export var speed: float = 50.0
@export var hp: int = 3
@export var damage: int = 1
@export var exp_value: int = 1
@export var enemy_type: int = 0  # EnemyType enum

enum EnemyType { CHASER, SHOOTER, BOMBER, DASHER, BOSS }

var _target: Player
var _stun_timer := 0.0
```

**敌人行为模式：**
- CHASER: 追踪玩家，近战攻击
- SHOOTER: 保持距离，远程射击
- BOMBER: 快速冲向玩家，自爆
- DASHER: 间歇性冲锋
- BOSS: 多阶段，多种攻击模式

### 5. 强化升级系统

过关后三选一。

```gdscript
extends Resource
class_name UpgradeResource

@export var upgrade_name: String
@export var description: String
@export var icon: Texture2D
@export var rarity: int = 0  # Rarity enum
@export var modifiers: Array[Dictionary]
# modifiers = [
#   {stat="move_speed", value=0.15, op=1},
#   {stat="max_hp", value=2, op=0}
# ]
# op: 0=加法, 1=乘法, 2=覆盖

enum Rarity { COMMON, RARE, EPIC, LEGENDARY }
```

### 6. 经济与掉落

```
击杀敌人 → 金币/经验
金币 → 商店购买武器/恢复
拾取物规则：
  - 普通敌人: 30% 掉落金币
  - 精英: 100% 金币 + 概率道具
  - Boss: 钥匙 + 高品质武器
```

### 7. 自动加载 (Autoload)

```gdscript
extends Node
class_name GameState

var character_stats: Dictionary
var unlocked_items: Array[String]
var total_runs: int
var best_wave: int
var total_kills: int
```

```gdscript
extends Resource
class_name RunData

var current_hp: int
var max_hp: int
var gold: int
var keys: int
var upgrades: Array
var current_weapon: Resource
var cleared_rooms: Array[int]
var enemies_killed: int
var elapsed_time: float
```

## 项目结构

```
roguelike-shooter/
├── scene/
│   ├── Player/         # player.tscn + player.gd
│   ├── Enemy/          # enemy_chaser, shooter, bomber, boss
│   ├── Weapon/         # bullet.tscn + bullet.gd
│   ├── Room/           # room_base, combat, treasure, boss
│   ├── UI/             # hud, upgrade_select, pause, death_screen
│   └── Menu/           # main_menu, character_select
├── resources/
│   ├── weapons/        # WeaponResource .tres
│   ├── upgrades/       # UpgradeResource .tres
│   └── tilesets/
├── scripts/
│   ├── auto_load/      # game_state, audio_manager
│   └── utils/          # procedural_generator, math_utils
├── art/
├── sounds/
├── project.godot
└── export_presets.cfg
```

## 房间生成算法

```
1. 确定房间数量 (3-5战斗 + Boss + 可能商店)
2. 随机游走生成房间拓扑图
3. 走廊连接房间门
4. 标记起始/Boss房间
5. 只加载当前及相邻房间
```

```gdscript
func generate_dungeon(room_count: int, seed: int) -> Dictionary:
    var rng := RandomNumberGenerator.new()
    rng.seed = seed
    var rooms: Array[Rect2i] = [Rect2i(0, 0, 10, 8)]
    for i in range(room_count - 1):
        _attach_room(rooms, rng)
    return {rooms=rooms}
```

## 相机

```gdscript
# Camera2D: smoothing_enabled=true, smoothing_speed=5.0

func _shake(intensity: float, duration: float) -> void:
    var tween := create_tween()
    tween.tween_method(_apply_shake, intensity, 0.0, duration)

func _apply_shake(value: float) -> void:
    offset = Vector2(randf_range(-value, value), randf_range(-value, value))
```

## 掉落物拾取 (磁吸效果)

```gdscript
# Area2D 拾取物
func _physics_process(delta: float) -> void:
    if _magnetic:
        var player := get_tree().get_first_node_in_group("player") as Node2D
        if player and global_position.distance_to(player.global_position) < _magnet_range:
            global_position = global_position.move_toward(
                player.global_position, _magnet_speed * delta)
```

## 伤害数字

```gdscript
func show_damage_number(position: Vector2, amount: int, is_crit: bool = false) -> void:
    var label := Label.new()
    label.text = str(amount)
    label.global_position = position + Vector2(randf_range(-8, 8), 0)
    label.add_theme_font_size_override("font_size", is_crit ? 24 : 16)
    label.add_theme_color_override("font_color", is_crit ? Color.YELLOW : Color.WHITE)
    get_tree().current_scene.add_child(label)
    var tween := create_tween()
    tween.tween_property(label, "position:y", label.position.y - 30, 0.8)
    tween.tween_property(label, "modulate:a", 0.0, 0.8)
    tween.finished.connect(label.queue_free)
```

## 开发阶段

### Phase 1: 基础框架
- 项目初始化
- 玩家移动 + 射击 + 闪避
- 基础敌人 + TileMap 房间生成
- 房间切换

### Phase 2: 战斗系统
- 多种敌人类型
- 武器系统 (散射/穿透/元素)
- 碰撞伤害 + 无敌帧 + 屏幕震动
- 掉落物
- 波次生成

### Phase 3: 肉鸽循环
- 升级三选一 UI
- 经济 + 商店
- Boss 战
- 永久死亡 + 结算
- 全局解锁

### Phase 4: 打磨
- 音效管理
- 特效粒子
- 平衡调整
- 多角色
- 镜头优化

## 编码规范

遵循 `F:\CLAUDE.md` 所有规范：
- 强制类型标注
- `_` 前缀私有变量
- `@export` 在前，`@onready` 节点引用
- 信号用代码 `connect()`
- 场景用 `preload()` 预加载
