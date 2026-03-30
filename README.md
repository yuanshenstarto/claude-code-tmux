# claude-code-tmux

An [OpenClaw](https://openclaw.ai) skill that runs coding tasks using **Claude Code in a persistent tmux session with git worktree isolation**.

## What it does

- Every task gets its own **git worktree** (isolated branch, no file conflicts)
- Claude Code runs in a **tmux session** — persistent, multi-turn conversation
- The agent acts as a **relay**: forwards Claude Code's plans/questions to you, sends your answers back
- Claude Code always shows a **plan first** and waits for your approval before touching files

## Install

```bash
npx clawhub@latest install claude-code-tmux
```

Or clone directly:
```bash
git clone https://github.com/yuanshenstarto/claude-code-tmux.git
```

## Usage

Just ask your OpenClaw agent to do a coding task — it will automatically:

1. Create a worktree for the task
2. Open a tmux session and start Claude Code
3. Show you the plan before any files are changed
4. Relay questions and answers between you and Claude Code
5. Clean up when done

## Workflow

```
You → OpenClaw agent → tmux session (Claude Code) → git worktree
                  ↑________________________________↓
                      relay: plans, questions, answers
```

When finished, the branch is preserved so you can switch to it in your main workspace and test.

## Requirements

- [tmux](https://github.com/tmux/tmux)
- [Claude Code CLI](https://github.com/anthropics/claude-code) (`npm install -g @anthropic-ai/claude-code`)
- Node.js >= 20
- git

## License

MIT
