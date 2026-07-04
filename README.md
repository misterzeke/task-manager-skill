# 🎯 Task Manager — Claude Code 技能

基于 Claude Code 原生 `TaskCreate` / `TaskUpdate` / `TaskList` / `TaskGet` 工具的**轻量任务管理协议**。

```
~~📦 第一阶段：调研 ✅~~
  ~~🔍 信息收集完成 ✅~~
  ~~📝 文档输出完成 ✅~~
  🛠️ 第二阶段：实现 🟡 进行中
    1. 核心逻辑 ← 正在写
    2. 测试 ⏳ 被 #1 阻塞
    3. 部署 ⏳ 待开始
```

## ✨ 功能特点

- **多步骤追踪** — 把复杂任务拆成原子步骤，一目了然
- **依赖链** — `addBlockedBy` 确保执行顺序不乱
- **发现即记录** — 过程中发现的重要信息即时存档，不等做完再补
- **完成元数据** — 每次 `completed` 都附带总结、文件列表、时间戳
- **视觉进度** — 回复中用删除线 + emoji 展示层级进度
- **父子层级** — 命名缩进惯例实现视觉嵌套，系统不依赖父任务

## 📦 安装

```bash
# 克隆到 Claude Code 技能目录
cd .claude/skills/
git clone https://github.com/misterzeke/task-manager-skill.git task-manager

# 或者手动下载 SKILL.md
mkdir -p .claude/skills/task-manager
# 把 SKILL.md 放进该目录
```

然后在 `CLAUDE.md` 中引用：

```markdown
### 任务管理
3 步以上的任务先用 TaskCreate 建任务列表，建之前加载 task-manager skill。
参见 `.claude/skills/task-manager/SKILL.md`。
```

## 🚀 使用方式

技能会在你说以下内容时自动触发：
- "建任务列表"
- "分成几步来做"
- 任何 3 步以上的多步骤任务

也可以手动调用：`/task-manager`

### 快速开始

```javascript
// 1. 建任务
TaskCreate({ subject: '📦 项目', activeForm: '规划中' })
TaskCreate({ subject: '  🅰️ 第一步', activeForm: '正在做A' })
TaskCreate({ subject: '  🅱️ 第二步', activeForm: '正在做B' })

// 2. 设同级依赖（不依赖父任务）
TaskUpdate({ taskId: '2', addBlockedBy: ['1'] })

// 3. 带元数据完成
TaskUpdate({
  taskId: '1',
  status: 'completed',
  metadata: {
    summary: '调研完成',
    filesModified: ['src/main.ts'],
    completedAt: new Date().toISOString(),
  },
})
```

## 📐 层级惯例

| 层级 | 缩进 | 示例 |
|:----:|:----:|:-----|
| 1️⃣ 父级 | 无缩进 | `📦 项目` |
| 2️⃣ 子级 | 2 空格 | `  🅰️ 模块` |
| 3️⃣ 孙级 | 4 空格 | `    1. 细项` |

**核心规则：** 父任务全程保持 `in_progress`，等所有子任务做完才标 `completed`——系统和视觉同步对齐。

## 📋 核心纪律

1. ✅ **完成必带元数据** — summary + filesModified + completedAt
2. ✅ **发现即记录** — 不等做完再补，即时写入 metadata
3. ✅ **做完查解锁** — 每完成一步调 TaskList() 看有没有解锁新任务
4. ✅ **父任务最后完成** — 所有子任务做完才关父任务
5. ✅ **视觉删除线标注** — 每次推进在回复里用删除线汇报进度

## 🔗 参考

- 基于 Claude Code 2.1.16+ 原生 Task 工具
- 灵感来自 [oimiragieo/agent-studio](https://github.com/oimiragieo/agent-studio) 的 task-management-protocol
- 和 [yonatangross/orchestkit](https://github.com/yonatangross/orchestkit) 的 task-dependency-patterns

## 📄 许可证

MIT — 自由使用、修改、分享。
