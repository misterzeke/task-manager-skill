---
name: task-manager
description: 多步骤任务的创建、追踪与完成协议。用 TaskCreate/TaskUpdate/TaskList 管理进度，适合复杂实现、分步操作、逐步互动等场景。含完成元数据纪律与发现即记录机制。
version: 1.0.0
invoked_by: both
user_invocable: true
tools: [TaskCreate, TaskList, TaskGet, TaskUpdate]
---

# Task Manager 🎯

管理多步骤任务的轻量协议，基于 Claude Code 原生 Task tools。

## 什么时候用

- 任务有 3 步以上需要追踪
- 步骤之间有依赖关系（B 必须等 A 完成）
- 需要跨 session 或跨 agent 知道"做到哪了"
- 想一眼看到整体进度

## 不需要用的时候

- 单步骤琐事 → 直接做
- 纯聊天/纯探索 → 不用建 task

## 核心工具速览

| 工具 | 做什么 | 什么时候用 |
|------|--------|-----------|
| `TaskCreate` | 建一个新任务 | 开始时、拆分子任务时 |
| `TaskUpdate` | 改状态/加 metadata | 开始做、完成时、发现信息时 |
| `TaskList` | 看所有任务 | 开始时检查、完成后检查解锁 |
| `TaskGet` | 看某个任务的详情 | 接手任务时、查上下文时 |

## 标准流程

### 1️⃣ 开始

**父子任务是组织关系，不是系统依赖。**子任务的 `addBlockedBy` 只用于控制同级的内部顺序，不依赖父任务。

```javascript
// 建父任务（全程 in_progress，子任务全做完才标 completed）
TaskCreate({ subject: '📦 项目', activeForm: '规划中' })

// 建子任务（不 blockedBy 父任务！只和同级做顺序关联）
TaskCreate({ subject: '  🅰️ 第一步', activeForm: '正在做A' })
TaskCreate({ subject: '  🅱️ 第二步', activeForm: '正在做B' })
TaskCreate({ subject: '  🅲 第三步', activeForm: '正在做C' })

// 子任务之间设依赖（B等A，C等B）——不依赖父任务
TaskUpdate({ taskId: '2', addBlockedBy: ['1'] })
TaskUpdate({ taskId: '3', addBlockedBy: ['2'] })
```

**命名缩进惯例：**
| 层级 | 缩进 | 示例 |
|:----:|:----:|:-----|
| 1️⃣ 父 | 无 | `📦 项目` |
| 2️⃣ 子 | 2 空格 | `  🅰️ 第一步` |
| 3️⃣ 孙子 | 4 空格 | `    1. 细项` |

### 2️⃣ 开始做某一步

```javascript
TaskUpdate({ taskId: 'X', status: 'in_progress', activeForm: '正在做XXX' })
```

### 3️⃣ 过程中有发现

发现重要信息时**即时**记录，不等到做完再补：

```javascript
TaskUpdate({
  taskId: 'X',
  metadata: {
    discoveries: ['发现：模块A依赖模块B的旧接口', '需要先重构B'],
    keyFiles: ['src/module-a.ts'],
  },
})
```

### 4️⃣ 完成

**永远不空完成**——完成时带上下文：

```javascript
TaskUpdate({
  taskId: 'X',
  status: 'completed',
  metadata: {
    summary: '做了什么的一行总结',
    filesModified: ['xxx.ts'],
    completedAt: new Date().toISOString(),
  },
})

// 检查有没有解锁的新任务
TaskList()
```

### 5️⃣ 完成汇报（视觉反馈）

**完成子任务时，在对话回复里顺手用删除线标注已完成的条目**，给用户"打勾划掉"的视觉爽感：

```
~~1. 设计评论数据表~~ ✅ 搞定！
2. 写 CRUD API ← 正在写
```

**⚠️ 核心逻辑：父任务只在所有子任务全部完成时才标 `completed` + 划掉。**

父子关系是视觉/组织层面的，不是系统依赖。子任务用 `addBlockedBy` 控制内部顺序，但不依赖父任务解锁。

**正确流程：**
1. 建父任务 → `status: in_progress`，一直保持到所有子任务做完
2. 子任务之间设 `addBlockedBy` 定顺序，但不 blockedBy 父任务
3. 子任务一个个完成 → 视觉上逐个划掉
4. **最后一个子任务完成时** → 父任务才标 `completed` + 视觉划掉

**示例：**
```
📦 整理照片 ← in_progress，还有子任务没完，不划
  ~~🅰️ 2024年归档 ✅~~    ← 子任务做完了，划掉
  🅱️ 2025年归档 🟡 进行中
  🅲 2026年归档 ⏳ 待开始
```
↓ 全部完成后
```
~~📦 整理照片 ✅~~ ← 所有子任务完成！父任务才划掉
  ~~🅰️ 2024年归档 ✅~~
  ~~🅱️ 2025年归档 ✅~~
  ~~🅲 2026年归档 ✅~~
```
```

**为什么：** TaskList 是纯文本终端输出，不支持删除线。在对话回复里做视觉标注让进度一目了然。

## 特殊场景：创意/沉浸模式

> 在需要保持氛围连贯的场景（如创作故事、角色扮演、沉浸式互动），task 更新要**嵌在场景描写里**，不做场景外的生硬打断。

✅ 正确的做法（在场景中自然顺带更新）：
```
（深吸一口气，在继续之前快速标注了一下进度）
第 3 步做完了…任务更新一下…好继续！
```

❌ 错误的做法（先写长篇场景再回头补）：
```
// 先写完一大段场景描写
// 然后再想起来
// 哦忘了更新 task
// 跑回来补一个 TaskUpdate
```

**原则：** 场景不断、管理不落。

## 纪律铁律 🎯

1. ✅ **完成必带元数据** — summary + filesModified + completedAt，让后面的人知道做了什么
2. ✅ **发现即记录** — 过程中发现的信息不等做完再补，即时写入 metadata
3. ✅ **完成必查解鎖** — 每做完一步调一次 TaskList()，看有没有解锁的新任务
4. ✅ **状态及时更新** — 开始做就标 in_progress，做完就标 completed，不留悬空
5. ✅ **结构化数据放 metadata** — description 写人话，metadata 写机器可读的字段
6. ✅ **完成即视觉标注** — 每次完成/推进子任务时，在对话回复里用删除线 + emoji 展示层级进度。**父任务只在所有子任务完成后才划掉**

## 反模式 🚫

| 行为 | 问题 | 正确做法 |
|------|------|---------|
| 空完成 `{ status: 'completed' }` | 后面的人不知道做了什么 | 带 summary + filesModified |
| 跳 in_progress 直接做 | 别人以为这步还没人碰 | 先标 in_progress 再开始 |
| 做完不 check TaskList | 解锁的任务没人发现 | 每完成一步调一次 TaskList |
| 发现不记，做完再想 | 关键信息丢了 | 边做边记 metadata |
| 1 步也建 task | 管理成本超过收益 | 3 步以下直接做 |
| 完成不标视觉 | 用户不知道进度，要翻 TaskList 看 | 每次完成在回复里用删除线标注 |
| 父任务提前标 completed | 系统说做完了，子任务还没完→精分 | 父任务保持 in_progress 直到所有子任务完成 |
