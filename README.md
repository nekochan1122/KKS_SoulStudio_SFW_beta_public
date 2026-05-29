<div align="center">

# KKS_SoulStudio_SFW_beta

**SFW LLM roleplay controller for Koikatsu Sunshine CharaStudio / KKS Studio**  
**面向 KKS CharaStudio 的 SFW 大语言模型角色扮演控制插件**

![Beta](https://img.shields.io/badge/status-beta-orange)
![SFW](https://img.shields.io/badge/build-SFW-brightgreen)
![DeepSeek](https://img.shields.io/badge/API-DeepSeek%20only-blue)
![Closed Source](https://img.shields.io/badge/source-closed--source-lightgrey)

[![Demo video](https://img.youtube.com/vi/34Jtlg3nM2M/hqdefault.jpg)](https://youtu.be/34Jtlg3nM2M)

**Demo video / 演示视频:**  
https://youtu.be/34Jtlg3nM2M

[Download latest beta](https://github.com/nekochan1122/KKS_SoulStudio_SFW_beta_public/releases/latest)  
[下载最新 beta 测试包](https://github.com/nekochan1122/KKS_SoulStudio_SFW_beta_public/releases/latest)

</div>

---

KKS_SoulStudio_SFW_beta is an experimental **SFW LLM roleplay controller** for **Koikatsu Sunshine CharaStudio / KKS Studio**.

KKS_SoulStudio_SFW_beta 是一个面向 **Koikatsu Sunshine / KKS CharaStudio** 的实验性 **SFW 大语言模型角色扮演控制插件**。

This public repository is distributed as a **closed-source beta test package**. It contains only the compiled plugin, default resource files, and documentation. Source code, handover files, and internal development notes are not included.

本公开仓库以 **闭源 beta 测试包** 形式分发。仓库只包含编译后的插件、默认资源文件和说明书，不包含源码、handover 文件或内部开发笔记。

---

# 中文说明

## 当前状态

当前版本：

```text
KKS_SoulStudio_SFW_beta v0.11.0-beta.1
```

这是 beta 测试版，不是稳定正式版。

插件已经可以用于实际测试，主要功能包括：

- 和当前选中的 Studio 角色聊天
- 根据对话设置表情
- 控制眼睛开合、嘴巴开合
- 控制脸红和流泪档位
- 根据对话触发身体动作
- 从 KKS 角色卡读取默认人设
- 支持角色书和世界书
- 支持纯 Chat 模式
- 使用内部 pose blend 缓解 KKS Studio 动作硬切
- 每次游戏启动生成一个完整 LLM session log

仍在调试中的部分：

- 动作选择稳定性
- 动作绑定覆盖率
- pose blend 在极端姿势下的表现
- 长时间游玩稳定性
- UI 易用性

---

## 需求

你需要：

- Koikatsu Sunshine / KKS CharaStudio
- BepInEx 5
- KKSAPI
- DeepSeek API key
- 网络连接

重要 API 兼容性说明：

```text
当前 beta 只正式支持 DeepSeek API。
其他 API、OpenAI-compatible 网关、本地模型、反代、第三方 provider 都还没有测试。
它们可能在某些情况下能跑，但本 beta 不保证支持。
```

建议测试时使用默认：

```text
Base URL = https://api.deepseek.com
```

---

## 安装方法

1. 从 Release 下载最新 zip。
2. 解压 zip。
3. 把里面的 `BepInEx` 文件夹复制到 KKS 游戏根目录。
4. 确认目录结构类似这样：

```text
Koikatsu Sunshine/
  BepInEx/
    plugins/
      KKS_SoulStudio_SFW_beta.dll
      KKS_SoulStudio/
        animations/
          actions.json
        expressions/
          brows.json
          eyes.json
          mouth.json
        personas/
          妮露.json
        worlds/
          须弥祖拜尔剧场.json
```

注意：虽然 DLL 叫 `KKS_SoulStudio_SFW_beta.dll`，但资源目录暂时仍然叫：

```text
BepInEx/plugins/KKS_SoulStudio/
```

这是为了保持 beta 内部路径兼容，降低资源读取出错的风险。

5. 启动 CharaStudio。
6. 按 `F1`，找到 `KKS_SoulStudio_SFW_beta`。
7. 在 API 设置里填入 DeepSeek API key。
8. 在 Studio 中选中一个角色。
9. 按 `F8` 打开或关闭聊天窗口。

---

## 基础使用

1. 在 Studio 中选中一个角色。
2. 按 `F8` 打开插件窗口。
3. 如果角色未启用插件，在窗口中启用。
4. 输入消息。
5. 点击发送。

测试用例：

```text
你好，先自然站好。
```

```text
慢慢走两步。
```

```text
蹲下来看看地面。
```

```text
有点不好意思地低下头。
```

```text
拿起一本音乐史的书认真读几页，不要听音乐。
```

插件会让 LLM 生成角色台词，并在需要时调用表情工具和动作工具。

---

## API 设置

打开：

```text
F1 -> KKS_SoulStudio_SFW_beta -> API
```

主要设置：

- `DeepSeek API key`  
  填入你的 DeepSeek API key。

- `Base URL`  
  默认 `https://api.deepseek.com`。普通测试不要改。非 DeepSeek endpoint 暂不支持。

- `Model`  
  模型名称。建议先使用 DeepSeek 官方模型测试。

- `Temperature`  
  越高越随机，越低越稳定。

- `Max tokens`  
  控制回复长度上限。

- `Timeout`  
  网络慢或模型慢时可以适当调高。

- `Prompt post-processing`  
  普通用户建议保持 `None`。Merge/Semi/Strict/Single 类模式会改变消息结构，可能影响连续 tool-only 回合。

---

## 动作系统概览

本 beta 已经废弃旧的固定 `action_id` 方式。现在 LLM 选择的是：

```text
category + variant + intensity
```

例子：

```text
posture / squat / 3
move / walk / 2
activity / study / 2
shy / hide_face / 3
```

插件再从这里查找对应的 KKS Studio 动作：

```text
BepInEx/plugins/KKS_SoulStudio/animations/actions.json
```

当前大类包括：

```text
none
idle
move
posture
greet
agree
deny
happy
shy
sad
angry
surprised
think
tired
activity
```

当前测试包里 `actions.json` 已经有默认绑定，不是空模板。  
已知 `agree` 和 `deny` 暂时可能还是占位，点头/摇头动作后续会继续补。

---

## 如何绑定动作

动作绑定文件位置：

```text
BepInEx/plugins/KKS_SoulStudio/animations/actions.json
```

一个动作大类的结构大致如下：

```json
{
  "id": "move",
  "label": "移动",
  "description": "走路、跑步、靠近、离开。",
  "cooldown_turns": 3,
  "levels": [],
  "variants": [
    {
      "id": "walk",
      "label": "行走",
      "description": "走路、走两步、靠近或离开。",
      "levels": [
        {
          "level": 2,
          "candidates": [
            {
              "group": 0,
              "category": 3,
              "no": 0,
              "name": "Walking 1"
            },
            {
              "group": 0,
              "category": 3,
              "no": 1,
              "name": "Walking 2"
            }
          ]
        }
      ]
    }
  ]
}
```

字段说明：

- `id`  
  动作大类或 variant 的内部 ID。不要随便改已有 ID，否则 prompt 和宿主修正规则可能对不上。

- `label`  
  给人看的名字。

- `description`  
  给 LLM 和维护者看的说明。

- `cooldown_turns`  
  冷却回合数，防止同一个动作连续反复触发。

- `levels`  
  这个大类直接绑定的强度层级。

- `variants`  
  细分动作。例如 `move` 下面有 `walk` 和 `run`，`posture` 下面有 `stand`、`sit`、`squat` 等。

- `level`  
  强度等级，范围通常是 `1..5`。LLM 输出 intensity 后，插件会选择最接近的 level。

- `candidates`  
  这个 level 下可以随机触发的具体 Studio 动作列表。

- `group` / `category` / `no`  
  KKS Studio 动作的三个编号，对应 dump 里的三列。

- `name`  
  备注名，只用于阅读和日志，不影响播放。

### 一个 action 可以绑定多个动作吗？

可以。

同一个 `level` 的 `candidates` 里可以放多个动作，插件会随机选择一个。

例如：

```json
{
  "level": 2,
  "candidates": [
    { "group": 0, "category": 3, "no": 0, "name": "Walking 1" },
    { "group": 0, "category": 3, "no": 1, "name": "Walking 2" }
  ]
}
```

当 LLM 选择：

```text
move / walk / 2
```

就会在 `Walking 1` 和 `Walking 2` 之间随机触发。

### 如何从 dump 里找动作？

1. 打开插件窗口。
2. 打开调试工具。
3. 使用 dump 动作按钮。
4. 插件会把 Studio 动作列表导出到：

```text
BepInEx/plugins/KKS_SoulStudio/animations/dump.txt
```

dump 里通常会有类似：

```text
0    3    0    Walking 1
0    3    2    Running 1
0    8    0    Squatting 1
0    8    8    Girly Sitting
```

这四列含义是：

```text
group    category    no    name
```

把前三列填入 `actions.json` 的 candidate：

```json
{
  "group": 0,
  "category": 8,
  "no": 0,
  "name": "Squatting 1"
}
```

### 绑定到哪里？

常见映射建议：

```text
走路       -> move / walk
跑步       -> move / run
站立       -> posture / stand 或 idle / stand
坐下       -> posture / sit
可爱坐姿   -> posture / girly_sit
蹲下       -> posture / squat
躺下       -> posture / lie
挥手       -> greet / wave
害羞低头   -> shy / hide_face
脸红扭捏   -> shy / embarrassed
读书       -> activity / study
吃东西     -> activity / eat
喝水       -> activity / drink
画画       -> activity / draw
听音乐     -> activity / listen_music
睡觉       -> tired / sleep
```

### 修改后如何生效？

修改 `actions.json` 后：

1. 确认 JSON 格式正确。
2. 回到插件窗口。
3. 点击重载动作。
4. 或者重启 CharaStudio。

如果 JSON 格式写错，插件可能无法加载动作目录。遇到问题先看：

```text
BepInEx/LogOutput.log
BepInEx/plugins/KKS_SoulStudio/logs/llm_session_*.log
```

### 动作绑定注意事项

- 不要删除 `actions` 根数组。
- 不要把 `group/category/no` 写成字符串，应该是数字。
- 不要随便改已有 `category id` 和 `variant id`。
- 可以增加候选动作，也可以调整 level。
- 同一个动作可以出现在多个 action 或 variant 里。
- 强烈建议先备份 `actions.json` 再大改。
- 动作越多不一定越好，质量和分类准确性更重要。

---

## 表情系统

表情由这些部分组成：

- brows tag
- eyes tag
- mouth tag
- eyes_open
- mouth_open
- face_blush
- tears

`face_blush` 和 `tears` 在 KKS 里不是连续滑杆，本插件按档位处理：

```text
0 = 无
1 = 轻微
2 = 明显
3 = 强烈
```

这意味着模型不需要为了哭泣强行选择 `cry` 眼型，也不需要为了脸红强行选择 blush 嘴型。  
比如可以使用 worried/sad 眼型，同时设置 `tears=1..3`。

---

## 角色卡、角色书和世界书

如果没有载入角色书，插件会读取 KKS 角色卡里的基础信息作为默认人设。

可能读取的信息包括：

- 姓名
- 昵称
- 性格
- 生日
- 血型
- 特质
- profile 文本

示例角色书和世界书位置：

```text
BepInEx/plugins/KKS_SoulStudio/personas/妮露.json
BepInEx/plugins/KKS_SoulStudio/worlds/须弥祖拜尔剧场.json
```

本 beta 追求简单易用，不要求玩家手动设置复杂 lorebook key。  
小书会整体注入，大书会由插件自动选取相关片段。

编辑书后，可以在插件窗口点击重载书，或重启 CharaStudio。

---

## 纯 Chat 模式

纯 Chat 模式会关闭：

- system prompt
- 角色卡 fallback
- 角色书
- 世界书
- 动作工具
- 表情工具

它适合测试模型原始聊天能力。  
如果你想让角色做动作和表情，请关闭纯 Chat 模式。

---

## 动作过渡 / Pose Blend

KKS Studio 的动作切换经常硬切。

本 beta 内置了一个 pose blend 过渡层：

1. 切动作前捕捉旧姿态骨骼。
2. 切到新 Studio 动作。
3. 在 `LateUpdate` 中把旧姿态逐帧混到新动画姿态。
4. 到达设定时间后释放控制，让新动画正常播放。

这不是 KKS 原生 crossfade，而是为 Studio 动作切 controller 场景做的 beta 级 workaround。

如果过渡太慢或姿态怪，可以调整：

```text
Actions -> Gesture fade time
```

推荐范围：

```text
0.4 - 0.8
```

---

## 日志

LLM 请求和响应日志位置：

```text
BepInEx/plugins/KKS_SoulStudio/logs/
```

每次游戏启动会生成一个 session log：

```text
llm_session_yyyyMMdd_HHmmss_fff.log
```

在文件里可以搜索：

```text
[0001 REQUEST]
[0002 RESPONSE]
perform_action
set_expression
Action.Normalized
latest user command override
```

反馈 bug 时请尽量提供：

- session log
- 你输入的原文
- 实际发生了什么动作或表情
- 你期待发生什么
- 使用的模型名
- 修改过的重要配置

---

## 常见问题

### 聊天窗口打不开

按 `F8`。确认你在 CharaStudio 里，并且 BepInEx 成功加载了插件。

### 提示 API key 缺失

打开：

```text
F1 -> KKS_SoulStudio_SFW_beta -> API
```

填入 DeepSeek API key。

### 会说话但不会动

检查：

- `Actions -> Enable LLM gestures`
- `BepInEx/plugins/KKS_SoulStudio/animations/actions.json`
- 动作是否被 cooldown 跳过
- 目标 action 是否真的有 candidates

### 会动但没有表情

检查：

- `Expressions -> Enable LLM expressions`
- `expressions/brows.json`
- `expressions/eyes.json`
- `expressions/mouth.json`

### 一直显示发送中

点击插件里的停止/重置按钮，然后检查：

```text
BepInEx/LogOutput.log
BepInEx/plugins/KKS_SoulStudio/logs/llm_session_*.log
```

### 动作选错

请保留 session log。  
本 beta 已经有宿主侧修正，但动作选择仍在调试。

---

## 已知问题

- 复杂多步骤命令目前会被压缩成每回合一个动作。
- pose blend 在极端姿势下可能有奇怪骨骼过渡。
- 有些 Studio 动作尚未绑定。
- `agree` / `deny` 目前可能是占位，点头/摇头后续补。
- 角色书和世界书较长时会增加延迟。
- UI 仍在打磨。
- 当前包只包含 SFW 功能。

---

## 未来计划

SFW 版计划：

- 更干净的 UI
- 首次运行自动检查资源
- 更强的多步骤动作队列
- 更好的 pose blend 骨骼过滤
- 更多默认动作绑定
- 根据情绪和姿态选择 idle
- 更稳的长期记忆控制
- 更清晰的模型预设
- 一键打开日志目录
- 更多示例角色书和世界书
- API 异常和 malformed response 的恢复增强

未来 NSFW 版规划：

当前 beta 是 SFW only。  
未来可能会做独立的 NSFW 版，但不会混在这个 SFW 包里偷偷更新。

可能的 NSFW 版方向：

- 独立 NSFW 开关
- 独立 prompt 和 action catalog
- 明确用户 opt-in
- 研究成人场景相关插件联动
- 更谨慎的状态控制
- SFW / NSFW 行为严格隔离
- 独立发布包，避免 SFW 用户误装 NSFW 功能

当前 beta 不包含任何 NSFW 功能。

---

## 隐私说明

插件会把 prompt context 发送给你配置的 LLM API endpoint。

可能发送的内容包括：

- 最近聊天记录
- 角色卡 fallback 信息
- 角色书和世界书文本
- 当前 mood / posture / last_action / expression 等运行状态

请不要把你不愿发送给 API provider 的私人信息写进聊天、角色书或世界书。

---

## 闭源 beta 说明

本仓库只用于二进制 beta 测试。

包含：

- 编译后的插件 DLL
- 默认动作目录
- 默认表情字典
- 示例角色书和世界书
- 说明文档

不包含：

- 源码
- handover
- 内部开发笔记
- 私有测试文件

未来是否开源会另行决定。当前 beta 是闭源测试包。

---

# English Guide

## Status

Current version:

```text
KKS_SoulStudio_SFW_beta v0.11.0-beta.1
```

This is a beta test build, not a stable release.

Current features:

- chat with the selected Studio character
- LLM-driven expressions
- eye and mouth openness control
- face blush and tears levels
- body actions selected from dialogue
- character-card fallback persona
- optional persona books and world books
- pure chat mode
- internal pose-blend transition for smoother KKS Studio animation changes
- one full LLM session log per game launch

Still being tuned:

- action selection stability
- action binding coverage
- pose blend quality on extreme poses
- long-session stability
- UI polish

---

## Requirements

You need:

- Koikatsu Sunshine / KKS CharaStudio
- BepInEx 5
- KKSAPI
- DeepSeek API key
- internet connection

API compatibility note:

```text
This beta currently only officially supports the DeepSeek API.
Other APIs, OpenAI-compatible gateways, local models, reverse proxies, or third-party providers have not been tested yet.
They may work in some cases, but they are not supported for this beta.
```

Recommended default:

```text
Base URL = https://api.deepseek.com
```

---

## Installation

1. Download the latest release zip.
2. Extract the zip.
3. Copy the included `BepInEx` folder into your KKS game folder.
4. Confirm the final structure:

```text
Koikatsu Sunshine/
  BepInEx/
    plugins/
      KKS_SoulStudio_SFW_beta.dll
      KKS_SoulStudio/
        animations/
          actions.json
        expressions/
          brows.json
          eyes.json
          mouth.json
        personas/
          妮露.json
        worlds/
          须弥祖拜尔剧场.json
```

The DLL is named:

```text
KKS_SoulStudio_SFW_beta.dll
```

The data folder is still named:

```text
BepInEx/plugins/KKS_SoulStudio/
```

This is intentional for beta compatibility.

5. Start CharaStudio.
6. Press `F1` and find `KKS_SoulStudio_SFW_beta`.
7. Fill in your DeepSeek API key.
8. Select a character in Studio.
9. Press `F8` to open or close the chat window.

---

## Basic Usage

1. Select a character in Studio.
2. Press `F8` to open the plugin window.
3. Enable the selected character if needed.
4. Type a message.
5. Send.

Example prompts:

```text
你好，先自然站好。
```

```text
慢慢走两步。
```

```text
蹲下来看看地面。
```

```text
有点不好意思地低下头。
```

```text
拿起一本音乐史的书认真读几页，不要听音乐。
```

The plugin asks the LLM for character dialogue and, when needed, expression and action tool calls.

---

## API Settings

Open:

```text
F1 -> KKS_SoulStudio_SFW_beta -> API
```

Important settings:

- `DeepSeek API key`  
  Your DeepSeek API key.

- `Base URL`  
  Default is `https://api.deepseek.com`. Do not change it for normal testing. Non-DeepSeek endpoints are not officially supported in this beta.

- `Model`  
  Model name. Use a DeepSeek model for normal beta testing.

- `Temperature`  
  Higher means more varied roleplay. Lower means more stable behavior.

- `Max tokens`  
  Maximum response length.

- `Timeout`  
  Increase this if your network or model is slow.

- `Prompt post-processing`  
  Leave it at `None` unless you know why you need SillyTavern-style message processing.

---

## Action System Overview

The beta no longer uses old fixed `action_id` output.

The LLM chooses:

```text
category + variant + intensity
```

Examples:

```text
posture / squat / 3
move / walk / 2
activity / study / 2
shy / hide_face / 3
```

The host then maps that abstract request to a concrete KKS Studio animation from:

```text
BepInEx/plugins/KKS_SoulStudio/animations/actions.json
```

Current categories:

```text
none
idle
move
posture
greet
agree
deny
happy
shy
sad
angry
surprised
think
tired
activity
```

The included `actions.json` already has default bindings.  
Known placeholder categories: `agree` and `deny` may not have playable nod/shake-head candidates yet.

---

## How To Bind Actions

Action binding file:

```text
BepInEx/plugins/KKS_SoulStudio/animations/actions.json
```

Basic structure:

```json
{
  "id": "move",
  "label": "Move",
  "description": "Walking, running, approaching, leaving.",
  "cooldown_turns": 3,
  "levels": [],
  "variants": [
    {
      "id": "walk",
      "label": "Walk",
      "description": "Walking or taking a few steps.",
      "levels": [
        {
          "level": 2,
          "candidates": [
            {
              "group": 0,
              "category": 3,
              "no": 0,
              "name": "Walking 1"
            },
            {
              "group": 0,
              "category": 3,
              "no": 1,
              "name": "Walking 2"
            }
          ]
        }
      ]
    }
  ]
}
```

Field meanings:

- `id`  
  Internal action or variant ID. Avoid changing existing IDs unless you know what you are doing.

- `label`  
  Human-readable name.

- `description`  
  Description for maintainers and the LLM prompt.

- `cooldown_turns`  
  Number of turns before the same action can be repeated.

- `levels`  
  Direct intensity levels for the category.

- `variants`  
  Sub-actions. For example, `move` has `walk` and `run`; `posture` has `stand`, `sit`, `squat`, etc.

- `level`  
  Intensity level, usually `1..5`. The plugin chooses the nearest level to the LLM intensity.

- `candidates`  
  Concrete Studio animations that can be randomly selected.

- `group` / `category` / `no`  
  The three KKS Studio animation IDs from the animation dump.

- `name`  
  Human-readable note for logs and editing. It does not affect playback.

### Can one action contain multiple animations?

Yes.

Put multiple entries under the same `candidates` list. The plugin randomly selects one.

Example:

```json
{
  "level": 2,
  "candidates": [
    { "group": 0, "category": 3, "no": 0, "name": "Walking 1" },
    { "group": 0, "category": 3, "no": 1, "name": "Walking 2" }
  ]
}
```

When the LLM chooses:

```text
move / walk / 2
```

the plugin randomly plays `Walking 1` or `Walking 2`.

### How to find animation IDs from the dump

1. Open the plugin window.
2. Open debug tools.
3. Use the dump animations button.
4. The plugin writes the dump to:

```text
BepInEx/plugins/KKS_SoulStudio/animations/dump.txt
```

Typical dump lines:

```text
0    3    0    Walking 1
0    3    2    Running 1
0    8    0    Squatting 1
0    8    8    Girly Sitting
```

Column meaning:

```text
group    category    no    name
```

Copy the first three numbers into a candidate:

```json
{
  "group": 0,
  "category": 8,
  "no": 0,
  "name": "Squatting 1"
}
```

### Where should actions go?

Recommended mapping:

```text
walk             -> move / walk
run              -> move / run
stand            -> posture / stand or idle / stand
sit              -> posture / sit
girly sitting    -> posture / girly_sit
squat            -> posture / squat
lie down         -> posture / lie
wave             -> greet / wave
shy look down    -> shy / hide_face
embarrassed pose -> shy / embarrassed
read or study    -> activity / study
eat              -> activity / eat
drink            -> activity / drink
draw             -> activity / draw
listen to music  -> activity / listen_music
sleep            -> tired / sleep
```

### How to reload after editing

After editing `actions.json`:

1. Make sure the JSON is valid.
2. Return to the plugin window.
3. Press reload actions.
4. Or restart CharaStudio.

If actions fail to load, check:

```text
BepInEx/LogOutput.log
BepInEx/plugins/KKS_SoulStudio/logs/llm_session_*.log
```

### Binding tips

- Do not delete the root `actions` array.
- Keep `group`, `category`, and `no` as numbers, not strings.
- Avoid renaming existing category and variant IDs.
- You can add candidates and adjust levels.
- The same Studio animation may appear in multiple actions.
- Back up `actions.json` before large edits.
- More actions are not always better. Good classification matters more than raw count.

---

## Expression System

Expressions are split into:

- brows tag
- eyes tag
- mouth tag
- eyes_open
- mouth_open
- face_blush
- tears

`face_blush` and `tears` are treated as levels:

```text
0 = none
1 = light
2 = visible
3 = strong
```

The model does not need to force a `cry` eye tag to trigger tears, or a blush mouth tag to trigger blush.

---

## Character Card, Persona Books, And World Books

If no persona book is loaded, the plugin reads basic data from the KKS character card.

Possible fallback fields:

- name
- nickname
- personality
- birthday
- blood type
- traits
- profile text

Example books:

```text
BepInEx/plugins/KKS_SoulStudio/personas/妮露.json
BepInEx/plugins/KKS_SoulStudio/worlds/须弥祖拜尔剧场.json
```

The beta is designed to be simple. Players do not need to manually tune lorebook keys. Small books are injected directly; larger books are auto-selected by the host.

---

## Pure Chat Mode

Pure Chat mode disables:

- system prompt
- character-card fallback
- persona book
- world book
- action tool
- expression tool

Use it only when testing raw model behavior. Keep it off for normal character control.

---

## Motion Blending / Pose Blend

KKS Studio often hard-cuts between animations.

This beta includes a pose-blend transition layer:

1. capture the old body pose
2. load the new Studio animation
3. blend old bone rotations into the new animated pose in `LateUpdate`
4. release control after the fade duration

This is not native KKS crossfade. It is a beta workaround for Studio animation controller swaps.

Try:

```text
Actions -> Gesture fade time
```

Suggested range:

```text
0.4 - 0.8
```

---

## Logs

LLM logs:

```text
BepInEx/plugins/KKS_SoulStudio/logs/
```

One session log per game launch:

```text
llm_session_yyyyMMdd_HHmmss_fff.log
```

Search for:

```text
[0001 REQUEST]
[0002 RESPONSE]
perform_action
set_expression
Action.Normalized
latest user command override
```

When reporting bugs, include:

- session log
- exact prompt
- actual action/expression
- expected action/expression
- model name
- important config changes

---

## Troubleshooting

### Chat window does not open

Press `F8`. Make sure BepInEx loaded the plugin and you are in CharaStudio.

### API key missing

Open:

```text
F1 -> KKS_SoulStudio_SFW_beta -> API
```

Fill in your DeepSeek API key.

### Character talks but does not move

Check:

- `Actions -> Enable LLM gestures`
- `BepInEx/plugins/KKS_SoulStudio/animations/actions.json`
- action cooldown
- whether the target action has candidates

### Character moves but has no expression

Check:

- `Expressions -> Enable LLM expressions`
- `expressions/brows.json`
- `expressions/eyes.json`
- `expressions/mouth.json`

### Stuck on sending

Use the stop/reset button, then check:

```text
BepInEx/LogOutput.log
BepInEx/plugins/KKS_SoulStudio/logs/llm_session_*.log
```

### Wrong action

Please keep the session log. The beta has host-side normalization, but action selection is still being tuned.

---

## Known Issues

- complex multi-step commands become one action per turn
- pose blend may look odd on extreme poses
- some Studio animations are not bound yet
- `agree` / `deny` may be placeholders for now
- long books can increase latency
- UI polish is not final
- this package is SFW only

---

## Roadmap

Planned SFW improvements:

- cleaner UI
- first-run resource checks
- stronger multi-step action queue
- better pose-blend bone filtering
- more default animation bindings
- emotion/posture-aware idle selection
- better long-term memory controls
- clearer model presets
- one-click log folder access
- more example persona/world books
- stronger recovery from API errors and malformed responses

Future NSFW edition plan:

The current beta is SFW only. A separate NSFW edition may be developed later as a separate optional package.

Possible NSFW goals:

- separate NSFW switch
- separate prompts and action catalog
- explicit user opt-in
- compatibility research for adult-scene plugins
- stricter state control
- clear SFW/NSFW separation
- independent package so SFW users do not receive NSFW features accidentally

No NSFW functionality is included in this beta.

---

## Privacy Notes

The plugin sends prompt context to your configured LLM API endpoint.

Possible prompt context:

- recent chat messages
- character-card fallback data
- persona/world book text
- runtime mood/posture/action/expression state

Do not put private information into chat, persona books, or world books unless you are comfortable sending it to your API provider.

---

## Closed-Source Beta Notice

Included:

- compiled plugin DLL
- default action catalog
- default expression dictionaries
- example persona/world books
- documentation

Not included:

- source code
- handover files
- internal development notes
- private test files

Source release may be reconsidered later. This beta is intentionally distributed as a closed-source test package.
