# Fire Emblem: Fortune's Weave — 60 fps without speedup
# 火焰纹章：万缕千丝 — 60fps 且不加速

**[English](#english) · [中文](#中文)**

---

## English

A game-side patch: every frame it writes 60 into the game's own *target frame rate* field, so the
game asks for present interval 1 and steps its logic by `60/target` (1.0 instead of 2.0).
Result: **60 fps with normal speed**.

> This is **not** Ctrl+U / `EDEN_NX2_SWAP_INTERVAL=1`. Those unlock the frame limiter and make the
> game run 2–3x too fast, because logic advances per presented frame.

- Game: `04007EF01F8AE000` v131072
- Emulator: Eglantine build-177 (nx2-46967a1b84), Eden-based
- Platform: Windows (Vulkan)

### Install

**1. Find the build id.** Start the game once, open `%APPDATA%\eden\log\eden_log.txt` and search for:

```
No cheats file found for title_id=04007EF01F8AE000, build_id=DB93B0125F485060
```

That 16-hex-character id is what the emulator uses for cheat file names. For v131072 the main
module is `DB93B0125F485060`.

**2. Put the file there.**

```
%APPDATA%\eden\load\04007EF01F8AE000\60fps\cheats\<build id>.txt
```

Example:

```
%APPDATA%\eden\load\04007EF01F8AE000\60fps\cheats\DB93B0125F485060.txt
```

A file named `cheat_*.txt` inside any mod folder also works.

**3. File content.**

```
[60fps]
40010000 00000000 0000003C
58000000 021E60D8
98000100 00000000 000001DC
A4100200 00000000
```

Do **not** add `#` comments: `TextCheatParser` rejects any character that is not whitespace,
`[name]` or a hex word, and the whole file is silently dropped.

**4. Restart and verify.** The log must show, and keep showing during gameplay:

```
nx2:vi2 present #1 swap interval 1 at 60 Hz (word 12 = 0x1003c)
```

If you still see `swap interval 2 (0x2003c)` after the first scene, the cheat is not applied.

### Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Still 30 fps, log shows `No cheats file found` | wrong build id or folder | rename to the id the log prints, keep the `cheats` subfolder |
| No cheat lines at all in the log | file not inside a mod folder | use `load/<title id>/<any folder>/cheats/<id>.txt` |
| `Failed to read cheats file` | `#` comments or stray text | keep only `[name]` + hex words |
| 60 fps but game runs fast | Ctrl+U / `EDEN_NX2_SWAP_INTERVAL` also on | turn those off |
| Movies skipped | PGO / MinGW builds of Eglantine | use the MSVC (standard) build |

### Recommended graphics settings (measured on a 5600X + RX 6800 XT)

| Setting | Value |
|---|---|
| Resolution | 1.5X (1620p) — 2X also viable if fps holds |
| VSync | Immediate (Off) |
| GPU Mode | **Fast** |
| GPU Fence Behavior | Default |
| Sync Memory Operations | Off |
| VRR | On |

Frame time: 16.7 ms in light scenes, 28–37 ms in heavy UI scenes. `GPU Mode: Fast` may cause
texture/effect artifacts — if so use `Accurate` and rely on VRR.

### How it works

The game's graphics object (global pointer at `[main+0x21e60d8]`) holds:

```
+0x1d8  display rate            (60)
+0x1dc  target frame rate       (60 menus / 30 gameplay)
+0x1e0  longest present interval (5)
```

The present asks for `min(+0x1e0, +0x1d8 / min(+0x1d8, +0x1dc))`, and 59 readers step the game by
`60.0 / target`. Writing 60 into `+0x1dc` gives interval 1 and a 1.0 step — 60 fps, same speed.
The cheat is just:

```
R1 = 60
R0 = *(u64*)(MAIN + 0x21E60D8)
R0 += 0x1DC
*(u32*)R0 = R1
```

Same mechanism is documented by nxnative for this title: `seedds/nxnative`,
`dm/games/04007EF01F8AE000/README.md`.

### If the game updates

Offsets are for v131072. Re-derive them by scanning `main` for the readers of `60.0 / target`
(nxnative's `dm/tools/fps_target.py`: `FPS_TARGET_PTR 0x21e60d8`, `FPS_TARGET_FIELD 0x1dc`), or
find the store that writes 30 when gameplay starts.

### Files

| File | Use |
|---|---|
| `DB93B0125F485060.txt` | ready-to-drop cheat for build-177 (rename when the build id changes) |
| `04007EF01F8AE000-60fps.yaml` | FPSLocker format, pointer chain |

### Credits

Mechanism: [seedds/nxnative](https://github.com/seedds/nxnative), `dm/games/04007EF01F8AE000/README.md`.
Emulator verification against Eglantine build-177 source.

---

## 中文

游戏侧补丁：每帧把游戏自己的「目标帧率」字段写成 60，游戏因此请求 present interval 1，
并把逻辑步进按 `60/target` 缩放到 1.0（原本 2.0）。结果是 **60fps 且速度正常**。

> 这**不是** Ctrl+U / `EDEN_NX2_SWAP_INTERVAL=1`：那两者会解除限速，让游戏以 2–3 倍速运行，
> 因为逻辑是跟着每一帧推进的。

- 游戏：`04007EF01F8AE000` v131072
- 模拟器：Eglantine build-177（nx2-46967a1b84），基于 Eden
- 平台：Windows（Vulkan）

### 安装

**1. 查 build id。** 先启动一次游戏，打开 `%APPDATA%\eden\log\eden_log.txt`，搜索：

```
No cheats file found for title_id=04007EF01F8AE000, build_id=DB93B0125F485060
```

这 **16 位十六进制**就是模拟器用来找金手指文件的 id。v131072 的 main 模块是 `DB93B0125F485060`。

**2. 放文件。**

```
%APPDATA%\eden\load\04007EF01F8AE000\60fps\cheats\<build id>.txt
```

例如：

```
%APPDATA%\eden\load\04007EF01F8AE000\60fps\cheats\DB93B0125F485060.txt
```

放在任意 mod 文件夹里、命名为 `cheat_*.txt` 也可以。

**3. 文件内容。**

```
[60fps]
40010000 00000000 0000003C
58000000 021E60D8
98000100 00000000 000001DC
A4100200 00000000
```

**不要**加 `#` 注释：`TextCheatParser` 只接受空白、`[名字]` 和十六进制字，其他字符会让整份文件被静默丢弃。

**4. 重启验证。** 日志里应出现，并在游戏中持续保持：

```
nx2:vi2 present #1 swap interval 1 at 60 Hz (word 12 = 0x1003c)
```

若第一个场景之后仍是 `swap interval 2 (0x2003c)`，说明补丁没生效。

### 排错

| 现象 | 原因 | 解决 |
|---|---|---|
| 仍是 30fps，日志有 `No cheats file found` | build id 或目录不对 | 改成日志里打印的 id，保留 `cheats` 子目录 |
| 日志完全没有 cheat 字样 | 文件不在 mod 文件夹内 | 用 `load/<title id>/<任意文件夹>/cheats/<id>.txt` |
| `Failed to read cheats file` | 注释或杂字符 | 只保留 `[名字]` 和十六进制行 |
| 60fps 但游戏变快 | 同时开了 Ctrl+U / `EDEN_NX2_SWAP_INTERVAL` | 关掉它们 |
| 过场被跳过 | Eglantine 的 PGO / MinGW 构建 | 换 MSVC 标准构建 |

### 推荐画质设置（Ryzen 5 5600X + RX 6800 XT 实测）

| 设置 | 值 |
|---|---|
| 分辨率 | 1.5X（1620p）；帧数够可以上 2X |
| 垂直同步 | Immediate（关） |
| GPU 模式 | **高速（Fast）** |
| GPU 围栏行为 | 系统默认 |
| 同步内存操作 | 关 |
| VRR | 开 |

帧时间：轻场景 16.7ms，重 UI 场景 28–37ms。`GPU 模式 = 高速` 可能出现贴图/特效异常，
出现时退回「精确（Accurate）」并靠 VRR 使用 45–60。

### 原理

游戏的图形对象（全局指针 `[main+0x21e60d8]`）里：

```
+0x1d8  显示刷新率      (60)
+0x1dc  目标帧率        (菜单 60 / 玩法 30)
+0x1e0  最长 present 间隔 (5)
```

present 请求 `min(+0x1e0, +0x1d8 / min(+0x1d8, +0x1dc))`，另有 59 处读取点按 `60.0 / target`
步进。把 `+0x1dc` 写成 60 即得到 interval 1 与 1.0 步进——60fps、速度不变。补丁内容就是：

```
R1 = 60
R0 = *(u64*)(MAIN + 0x21E60D8)
R0 += 0x1DC
*(u32*)R0 = R1
```

同一机制见 nxnative 的本项目文档：`seedds/nxnative`，`dm/games/04007EF01F8AE000/README.md`。

### 游戏更新后

偏移对应 v131072。游戏更新后用 nxnative 的 `dm/tools/fps_target.py`
（`FPS_TARGET_PTR 0x21e60d8`、`FPS_TARGET_FIELD 0x1dc`）扫描 `60.0 / target` 的读取点重新定位，
或找到"进入玩法时写 30"的那条 store。

### 文件

| 文件 | 用途 |
|---|---|
| `DB93B0125F485060.txt` | 已改好 id 的金手指，build-177 直接可用（换版本时改文件名即可） |
| `04007EF01F8AE000-60fps.yaml` | FPSLocker 格式（指针链） |

### 致谢

机制来源：[seedds/nxnative](https://github.com/seedds/nxnative)，
`dm/games/04007EF01F8AE000/README.md`。模拟器侧在 Eglantine build-177 源码上核对。
