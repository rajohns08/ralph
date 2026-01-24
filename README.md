# Ralph

An automated builder/reviewer loop that uses Claude for planning and implementation, and Codex for code review.

## Overview

Ralph breaks down a task into small, focused subtasks, then iteratively implements and reviews each one. This creates a feedback loop where:

1. **Planner** (Claude) - Explores the codebase and creates a task list
2. **Builder** (Claude) - Implements one task at a time
3. **Reviewer** (Codex) - Reviews the implementation and approves or rejects with feedback
4. If rejected, the builder addresses feedback and resubmits

This continues until all tasks are complete or the maximum iterations are reached.

## Requirements

- [Claude CLI](https://github.com/anthropics/claude-code) (`claude`)
- [Codex CLI](https://github.com/openai/codex) (`codex`)
- Bash
- Git (optional, for automatic commits)

## Installation

```bash
# Clone the repository
git clone <repo-url>
cd ralph

# Make executable (if not already)
chmod +x ralph

# Optionally add to PATH
cp ralph /usr/local/bin/
```

## Usage

### Start a new task

```bash
ralph "<prompt>" <max_iterations>
```

**Example:**
```bash
ralph "Add user authentication with JWT tokens" 10
```

### Resume an existing task

```bash
ralph --resume <max_iterations>
```

This continues from where you left off, using the existing `.ralph/tasks.md`.

### Check status

```bash
ralph --status
```

Shows the current task list and progress summary.

## How It Works

### Task States

| Symbol | State | Description |
|--------|-------|-------------|
| `[ ]` | Pending | Task not yet started |
| `[R]` | Ready for Review | Implementation complete, awaiting review |
| `[x]` | Complete | Approved by reviewer |
| `[!]` | Rejected | Needs fixes (see comments below task) |

### Workflow

```
┌─────────────┐
│   Planner   │  Creates task list
└──────┬──────┘
       │
       v
┌─────────────┐     ┌─────────────┐
│   Builder   │────>│  Reviewer   │
│  (Claude)   │<────│   (Codex)   │
└─────────────┘     └─────────────┘
    [ ] -> [R]         [R] -> [x] (approved)
    [!] -> [R]         [R] -> [!] (rejected)
```

### Files Created

Ralph creates a `.ralph/` directory containing:

- `tasks.md` - The task list with status markers
- `ralph.log` - Full execution log
- `state` - Internal state tracking

### Git Integration

If running in a git repository, Ralph automatically commits after each builder phase with the task description as the commit message.

## Example Session

```bash
$ ralph "Create a REST API for managing todos" 5

[2024-01-15 10:30:00] Starting Ralph
[2024-01-15 10:30:00] Prompt: Create a REST API for managing todos
[2024-01-15 10:30:00] Max iterations: 5

========================================
PLANNER PHASE - Creating Task List
========================================
...

========================================
BUILDER PHASE - Implementing Next Task
========================================
...

========================================
REVIEWER PHASE - Reviewing Implementation
========================================
...
```

## Tips

- **Start small**: Use a reasonable `max_iterations` (5-10) and increase if needed
- **Check status**: Use `--status` to see progress before resuming
- **Review the log**: Check `.ralph/ralph.log` for detailed execution history
- **Edit tasks manually**: You can edit `.ralph/tasks.md` to add, remove, or reorder tasks

## License

MIT
