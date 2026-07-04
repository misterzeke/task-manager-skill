# 🎯 Task Manager — Claude Code Skill

A lightweight task management protocol for **Claude Code**, built on the native `TaskCreate` / `TaskUpdate` / `TaskList` / `TaskGet` tools.

```
~~📦 Phase 1: Research ✅~~
  ~~🔍 Discovery complete ✅~~
  ~~📝 Documentation done ✅~~
  🛠️ Phase 2: Implementation 🟡 in progress
    1. Core logic ← working on it
    2. Tests ⏳ blocked by #1
    3. Deploy ⏳
```

## ✨ Features

- **Multi-step tracking** — break any complex task into atomic steps
- **Dependency chains** — `addBlockedBy` ensures correct execution order
- **Discoveries on the fly** — recording important findings as they happen, not after
- **Completion metadata** — every `completed` carries summary, files, timestamp
- **Visual progress** — strikethrough + emoji progress bars in responses
- **Parent-child hierarchy** — visual nesting via naming convention (no system dependency)

## 📦 Installation

```bash
# Clone into your Claude Code skills directory
cd .claude/skills/
git clone https://github.com/misterzeke/task-manager-skill.git task-manager

# Or just copy the SKILL.md manually
mkdir -p .claude/skills/task-manager
# Download SKILL.md into that folder
```

Then reference it in your `CLAUDE.md`:

```markdown
### Task Management
For tasks with 3+ steps, load task-manager skill first.
See `.claude/skills/task-manager/SKILL.md` for the full protocol.
```

## 🚀 Usage

The skill auto-activates when you say things like:
- "Create a task list"
- "Let's break this into steps"
- Any multi-step workflow with 3+ steps

Or invoke it manually: `/task-manager`

### Quick Start

```javascript
// 1. Create tasks
TaskCreate({ subject: '📦 Project', activeForm: 'Planning' })
TaskCreate({ subject: '  🅰️ Step 1', activeForm: 'Working on A' })
TaskCreate({ subject: '  🅱️ Step 2', activeForm: 'Working on B' })

// 2. Set internal dependencies (sibling tasks only, NOT parent)
TaskUpdate({ taskId: '2', addBlockedBy: ['1'] })

// 3. Complete with metadata
TaskUpdate({
  taskId: '1',
  status: 'completed',
  metadata: {
    summary: 'Research complete',
    filesModified: ['src/main.ts'],
    completedAt: new Date().toISOString(),
  },
})
```

## 📐 Hierarchy Convention

| Level | Prefix | Example |
|:-----:|:-------|:--------|
| 1️⃣ Parent | no indent | `📦 Project` |
| 2️⃣ Child | 2 spaces | `  🅰️ Module` |
| 3️⃣ Grandchild | 4 spaces | `    1. Task` |

**Key rule:** Parent tasks stay `in_progress` until ALL children are done. Only then mark parent `completed` — keeps system state and visual strikethrough in sync.

## 📋 Core Disciplines

1. ✅ **Always complete with metadata** — summary + filesModified + completedAt
2. ✅ **Record discoveries immediately** — don't wait until the end
3. ✅ **Check TaskList after each completion** — find newly unblocked tasks
4. ✅ **Parent = last to complete** — all children done before parent closes
5. ✅ **Visual strikethrough in replies** — gives that satisfying "checked off" feeling

## 🔗 Related

- Built on Claude Code 2.1.16+ native task tools
- Inspired by [oimiragieo/agent-studio](https://github.com/oimiragieo/agent-studio) (task-management-protocol) and [yonatangross/orchestkit](https://github.com/yonatangross/orchestkit) (task-dependency-patterns)

## 📄 License

MIT — free to use, modify, and share.
