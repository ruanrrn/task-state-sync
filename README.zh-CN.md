# Task State Sync

[English](README.md) | 简体中文

![Task State Sync banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-18324A?style=flat-square)
![Focus-State Sync](https://img.shields.io/badge/Focus-State%20Sync-8FD3C8?style=flat-square&labelColor=18324A)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-F6F4EE?style=flat-square&labelColor=355C7D)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DCEFE9?style=flat-square&labelColor=355C7D)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-F6F4EE?style=flat-square&labelColor=B45F3C)
![License-MIT](https://img.shields.io/badge/License-MIT-F6F4EE?style=flat-square&labelColor=18324A)

一个在多任务执行过程中维持连续性状态文件准确性的 OpenClaw 技能。

## Quick pitch

让 `TODO.md` 和 `memory/active-task.md` 在真实工作推进时保持同步。
状态漂移不是什么人格魅力，别等重启替你收尸。

## Why this exists

代理做事往往还行，但维护“之后还能接着做”的状态信息时，常常菜得很稳定。结果也很熟悉：`TODO.md` 过期，`memory/active-task.md` 还指着昨天的问题，已经完成的任务像鬼一样挂在队列里，系统一重启，现场直接变考古。

`task-state-sync` 就是拿来阻止这种慢性失忆的。

它教代理在什么时候写持久化状态，什么内容该进 `TODO.md`，什么内容该进 `memory/active-task.md`，以及在优先级、阻塞、下一步动作变化时，如何把这两份文件继续对齐。

## Works independently

`task-state-sync` 刻意保持窄职责，但单独使用也完全成立。

如果你的主要问题是状态漂移，而不是调度策略，那它本身就够用了。即使没有任何配套技能，它也能帮你稳定做到：

- 让 `TODO.md` 保持准确
- 让 `memory/active-task.md` 保持准确
- 保存后续恢复需要的重要 ID
- 清掉陈旧或已完成的任务
- 降低重启后的混乱程度

它不依赖 `task-orchestrator` 或 `multi-task-continuity` 才能发挥价值；那些仓库只是搭配起来更顺手。

## Family role

在这组仓库里，`task-state-sync` 的角色是“连续性文件维护专员”。

当真正的问题是状态陈旧、关键 ID 丢失，或者 `TODO.md` 与 `memory/active-task.md` 漂移时，就该用它。
别因为 bug 发生在多任务场景里，就把它硬塞成调度器。

## What the skill teaches

这个技能会要求代理：

- 在状态发生实质变化时同步连续性文件，而不是每走一步就重写
- 把 `TODO.md` 作为按聊天维度维护的未完成任务队列
- 把 `memory/active-task.md` 作为唯一的优先恢复 scratchpad
- 在重要 ID 第一次出现时就记下来
- 删除陈旧或已完成的项目，而不是把历史垃圾继续堆在活动状态文件里
- 在重置或计划重启前，提前写好可恢复状态

## When to use it

以下场景适合用 `task-state-sync`：

- 代理同时处理多个跨消息任务
- 工作需要跨重启或跨 session 保持连续
- 执行过程中最高优先级发生变化
- 出现或解除阻塞
- 需要保存后续恢复会用到的重要 ID
- `TODO.md` 和 `memory/active-task.md` 有漂移风险

## Example behavior

### Example 1: 出现新的 blocker

一个后台运行失败了，并返回 session ID 和日志路径。

好的代理应该：

1. 在 `TODO.md` 里写入 blocker 和下一步诊断动作
2. 判断这个失败是否已经变成当前最高优先级任务
3. 如果是，就重写 `memory/active-task.md`，把它升成恢复优先任务
4. 把 session ID 和日志路径记到后续恢复一定找得到的位置

### Example 2: 紧急任务插队

用户在其他工作还在进行时，又发来了一个线上紧急问题。

好的代理应该：

1. 重写 `TODO.md` 里的当前聊天区块
2. 把旧任务降级为次要工作，而不是直接删掉
3. 把紧急问题提升到 `memory/active-task.md`
4. 更新重启后的恢复提示语，让它反映真实情况

### Example 3: 任务完成

一个原本活跃的发布流程成功结束。

好的代理应该：

1. 从 `TODO.md` 中移除已完成项
2. 如果还有其他任务，重写剩余队列
3. 如果已经没有最高优先级任务，就清空 `memory/active-task.md`

## Related skills

这些是相关技能，不是依赖项：

- `task-orchestrator`：负责调度、优先级和分阶段进度策略 - <https://github.com/ruanrrn/task-orchestrator>
- `multi-task-continuity`：把调度、状态同步和重启恢复打包成完整工作流 - <https://github.com/ruanrrn/multi-task-continuity>

如果你的核心痛点就是状态漂移，这个仓库单独用就够了。

## Social preview

建议的社交预览图：`assets/social-preview.svg`

建议的一行文案：

> 让 `TODO.md` 和 `memory/active-task.md` 在真实工作推进时保持同步。

GitHub 说明：

- 当前 `gh` CLI 和 GraphQL `UpdateRepositoryInput` 都不支持写入自定义 social preview。
- 如果要把这张图设为仓库社交预览图，需要手动在仓库设置页面上传 `assets/social-preview.svg`。

## What you get

- `task-state-sync/` - 技能源码
- `dist/task-state-sync.skill` - 可直接导入的打包产物

## Install

可任选一种方式：

1. 将 `dist/task-state-sync.skill` 导入 OpenClaw 环境。
2. 如果你需要可编辑源码，就把 `task-state-sync/` 复制到你的 skills 目录。

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

详见 `CONTRIBUTING.md`，里面写了贡献范围、PR 期望，以及如何避免把这个仓库做成一坨泛化编排理论。

## Release hygiene

- 只要技能本体有实质变更，就重新生成 `dist/task-state-sync.skill`
- 仓库应保持聚焦在状态同步，而不是扩展成泛化编排策略
- README 里的示例必须和 `task-state-sync/SKILL.md` 的真实规则保持一致
- 对外展示要干净、双语、且单仓库即可自解释

## Repository

- GitHub: `https://github.com/ruanrrn/task-state-sync`
- License: MIT
