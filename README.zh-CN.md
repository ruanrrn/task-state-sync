# Task State Sync

[English](README.md) | 简体中文

![Task State Sync banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-18324A?style=flat-square)
![Focus-State Sync](https://img.shields.io/badge/Focus-State%20Sync-8FD3C8?style=flat-square&labelColor=18324A)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-F6F4EE?style=flat-square&labelColor=355C7D)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DCEFE9?style=flat-square&labelColor=355C7D)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-F6F4EE?style=flat-square&labelColor=B45F3C)
![License-MIT](https://img.shields.io/badge/License-MIT-F6F4EE?style=flat-square&labelColor=18324A)

一个用于在多任务执行过程中持续保持 `TODO.md` 与 `memory/active-task.md` 准确性的 OpenClaw skill。

## Overview

`task-state-sync` 是一个专门维护连续性状态文件的窄职责 skill，不是通用编排框架。

它负责在优先级变化、阻塞出现、后台运行启动、任务完成等真实执行过程中，让两份关键连续性文件始终和现实对齐：

- `TODO.md`：按聊天维度维护的持久未完成队列
- `memory/active-task.md`：唯一的“恢复时优先继续”的 scratchpad

这个 skill 故意做得很小。它不负责决定调度策略，不接管大范围工作流策略，也不替代更高层的协同技能。它存在的意义很直接：别让状态漂移把一次普通中断升级成事故复盘。

## Why this exists

很多代理会做事，但不太会在做事的同时维护“之后还能接着做”的状态。常见翻车姿势稳定得像 CI：

- `TODO.md` 落后于真实任务队列
- `memory/active-task.md` 指向错误的最高优先级任务
- 重要 ID 第一次出现时没有及时记住
- 已完成的工作还挂在活动列表里装死
- 一次重启就把活跃任务变成现场重建

`task-state-sync` 的目标，就是把连续性文件维护从“也许记得做”变成明确流程。

## Scope

当问题核心是“执行中的任务状态准确性”时，就该用这个仓库。

适合这些场景：

- 工作跨越多条消息，活跃队列会变化
- 执行过程中优先级或阻塞发生了实质变化
- 需要保存后续恢复一定会用到的重要 ID
- 如果不先写好连续性文件，重启或 session reset 会很痛
- `TODO.md` 与 `memory/active-task.md` 有明显漂移风险

不适合这些场景：

- 决定调度策略
- 提供通用任务优先级框架
- 单独负责重启自动恢复系统
- 承担整套多任务 operating model

如果主要问题是编排，去用配套仓库；如果主要问题是状态文件过期，这里才是正门。

## What the skill covers

`task-state-sync` 教代理在真正关键的时刻维护连续性文件：

- 只在状态发生实质变化时同步，而不是每个小动作都重写
- 把按聊天维度的队列写进 `TODO.md`
- 把唯一需要优先恢复的任务写进 `memory/active-task.md`
- 在重要 ID 第一次出现时立即记录
- 区分“被阻塞”与“只是等待中”
- 删除已完成或陈旧条目，而不是把假信息继续留在现场
- 在 reset 或计划重启前，提前写好可恢复状态

这个仓库刻意保持边界清晰：它标准化的是连续性文件维护，不是整个多任务系统。

## Workflow summary

一次典型的 `task-state-sync` 执行应该长这样：

1. 先重建真实状态：当前有哪些活跃任务、谁是最高优先级、哪些是阻塞、下一步是什么、哪些 ID 之后会用到。
2. 更新 `TODO.md`，让当前聊天的未完成队列反映真实情况。
3. 如果存在明确的最高优先级任务，就更新 `memory/active-task.md`。
4. 清掉陈旧或已完成项目，并确保两份文件里的重要 ID 不互相打架。
5. 再检查一次：下一步动作、阻塞状态、恢复提示语是否还和现实一致。

这听起来像卫生工作，但决定恢复能不能成功的，往往就是这点卫生工作。

## When to use it

当你需要让连续性文件在执行过程中仍然可信时，就用 `task-state-sync`。

典型触发点包括：

- 新任务插队并抢走最高优先级
- 某个 blocker 出现或解除
- 一个后台运行启动并返回值得保存的 ID
- 一个长任务需要干净的重启恢复信息
- 下一步具体动作已经变化
- 一个已完成任务应该从活动队列里消失

## Representative outcomes

### 阻塞信息落盘

一个后台运行失败了，并返回 session ID 和日志路径。

靠谱的代理应该立刻更新 `TODO.md`，判断这个失败是否已经成为最高优先级任务；如果是，就把它提升进 `memory/active-task.md`，并立即保存后续恢复需要的 ID。

### 优先级切换

在其他工作仍在进行时，新的紧急请求插了进来。

靠谱的代理应该重写当前聊天的任务队列，让旧任务继续被跟踪但降级处理，同时重写 `memory/active-task.md`，把新任务变成恢复时优先继续的主线。

### 完成后清理

一个之前处于活跃状态的任务成功完成。

靠谱的代理应该把它从 `TODO.md` 中移除，刷新剩余队列；如果已经没有明确的最高优先级任务，就清空 `memory/active-task.md`。

## Related skill repos

这些仓库是相关示例，不是必需依赖：

- `task-orchestrator`：负责调度、优先级和分阶段进度策略 - <https://github.com/ruanrrn/task-orchestrator>
- `multi-task-continuity`：更完整的工作模型，组合了编排、状态同步和可恢复连续性 - <https://github.com/ruanrrn/multi-task-continuity>

当失败模式是状态陈旧时，从这里开始；如果你还需要外围的工作流策略，再去用更大的仓库。

## Install

两种方式都可以：

1. 将 `dist/task-state-sync.skill` 导入 OpenClaw 环境。
2. 如果你需要可编辑源码，就把 `task-state-sync/` 复制到你的 skills 目录。

## What this repo contains

- `task-state-sync/` - skill 源码
- `dist/task-state-sync.skill` - 可直接导入的打包产物
- `assets/social-preview.svg` - 仓库 banner 与建议使用的 social-preview 资源

## Social preview

建议使用的 social preview 资源：`assets/social-preview.svg`

建议的一句话文案：

> Keep `TODO.md` and `memory/active-task.md` accurate while multitask work is still in flight.

GitHub 说明：

- 当前 `gh` CLI 和 GraphQL `UpdateRepositoryInput` 都不支持写入自定义 social preview。
- 如果要把这张图真正设成仓库 social preview，需要在仓库设置页面手动上传 `assets/social-preview.svg`。

## Repository layout

```text
task-state-sync/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── task-state-sync/
│   └── SKILL.md
└── dist/
    └── task-state-sync.skill
```

## Contributing

详见 `CONTRIBUTING.md`。里面写明了贡献范围、PR 预期，以及如何保持这个仓库继续聚焦连续性文件维护，而不是一路长歪成泛化编排策略。

## Release hygiene

- 只要 skill 本体发生实质变更，就重新生成 `dist/task-state-sync.skill`
- 保持 `README.md`、`README.zh-CN.md` 与 `task-state-sync/SKILL.md` 一致
- 持续保持这个仓库在技能家族中的窄职责定位：状态同步专员，不是总包
- 示例要保持操作性和诚实，不要把仓库描述得比实际更宽

## Repository

- GitHub: `https://github.com/ruanrrn/task-state-sync`
- License: MIT
