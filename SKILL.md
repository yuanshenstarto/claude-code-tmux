---
name: claude-code-task
description: Run coding tasks using Claude Code in a persistent tmux session with git worktree isolation. Activate when user asks to build, fix, refactor, review code, or run any multi-step coding task. Handles single and parallel tasks. Always uses tmux for persistent multi-turn conversation with Claude Code — never runs one-shot --print mode.
---

# Claude Code Task

Run coding tasks by spawning Claude Code in a **tmux session + git worktree**. Every task gets its own isolated branch and a persistent conversation session.

## Setup per task

```bash
# 1. Create worktree (always, even single task)
git -C <project> worktree add -b <branch> <worktree-path> main
# e.g. git -C ~/develop/myapp worktree add -b feat/login /tmp/task-login main

# 2. Symlink env files
ln -sf <project>/.env <worktree-path>/.env
ln -sf <project>/.env.local <worktree-path>/.env.local   # if exists

# 3. Create tmux session
tmux new-session -d -s <task-name> -c <worktree-path>

# 4. Start Claude Code (interactive, persistent, multi-turn)
tmux send-keys -t <task-name> "claude --dangerously-skip-permissions" Enter
```

## Plan mode (mandatory)

Always instruct Claude Code to show a plan first before making changes:

```
Your task prompt here.

Before making any changes, show me a plan of what you intend to do and wait for my approval.
```

When Claude Code outputs the plan, **relay it to the user** and wait for their confirmation before sending approval.

## Relaying messages

```bash
# Read Claude Code output
tmux capture-pane -t <task-name> -p | tail -30

# Send message to Claude Code
tmux send-keys -t <task-name> -l -- "<message>"
sleep 0.1
tmux send-keys -t <task-name> Enter

# Check if waiting for input (plan confirmation, permission, etc.)
tmux capture-pane -t <task-name> -p | tail -10 | grep -E "❯|Yes.*No|proceed|permission|plan|approve"
```

Relay flow:
1. Claude Code outputs a plan → relay to user
2. User says "ok" / "change X" → forward to Claude Code
3. Claude Code proceeds → monitor and relay any further questions

## Parallel tasks

Same pattern, multiple sessions:

```bash
tmux new-session -d -s task-a -c /tmp/task-a
tmux new-session -d -s task-b -c /tmp/task-b
# launch claude in each
```

Check all at once:
```bash
for s in task-a task-b; do
  echo "=== $s ==="
  tmux capture-pane -t $s -p 2>/dev/null | tail -5
done
```

## When task is done

```bash
# 1. Remove worktree (keeps the branch)
git -C <project> worktree remove <worktree-path>

# 2. User switches to branch in main workspace to test
# git switch <branch>

# 3. Kill tmux session
tmux kill-session -t <task-name>
```

## Node version

Use nvm if needed:
```bash
tmux send-keys -t <task-name> "nvm use 20 && claude --dangerously-skip-permissions" Enter
```

## Rules

- **Always** use worktrees — one per task, no exceptions
- **Always** use tmux — persistent session, multi-turn conversation
- **Never** use `--print` mode — that's stateless and loses context
- **Always** symlink `.env` files — don't copy
- **Always** ask Claude Code to show a plan first before touching any files
- **Always** relay the plan to user and wait for approval before proceeding
- One status message when starting, one when done or stuck
- See `references/troubleshooting.md` for common issues
