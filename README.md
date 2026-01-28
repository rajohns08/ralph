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
ralph "<prompt>" [max_iterations]
```

**Example:**
```bash
ralph "Add user authentication with JWT tokens"
ralph "Add user authentication with JWT tokens" 10
```

The `max_iterations` argument is optional and defaults to 100. This saves the prompt to `.ralph/prompt.txt` for future use.

Alternatively, create `.ralph/prompt.txt` first and run with just the iteration count:
```bash
mkdir -p .ralph
echo "Add user authentication with JWT tokens" > .ralph/prompt.txt
ralph 10
```

### Resume an existing task

```bash
ralph --resume [max_iterations]
# or simply
ralph [max_iterations]
```

This continues from where you left off, using the existing `.ralph/prompt.txt`. Iterations default to 100 if not specified.

### Pre-planned mode

If you want to create the task list yourself (or have Claude create it separately), you can skip the planning phases:

```bash
ralph --pre-planned [max_iterations]
ralph --pre-planned "<prompt>" [max_iterations]
```

**Requirements:**
- `.ralph/tasks.md` must already exist with your task list
- `.ralph/prompt.txt` must exist (or provide prompt as argument)

**Example:**
```bash
# Create your own task list
mkdir -p .ralph
cat > .ralph/tasks.md << 'EOF'
# Task List

## Tasks
- [ ] Set up project structure
- [ ] Add database models
- [ ] Implement API endpoints
- [ ] Add tests

## Task Legend
`[ ]` = Pending
`[R]` = Ready for review
`[x]` = Approved/Complete
`[!]` = Rejected (see comments below task)
EOF

echo "Build a todo API" > .ralph/prompt.txt

# Run with pre-planned tasks (skips planner and plan review phases)
ralph --pre-planned
```

This is useful when you want more control over the task breakdown or when resuming work where you've manually adjusted the tasks.

### Check status

```bash
ralph --status
```

Shows the current task list and progress summary.

### Dynamic Prompt Updates

The prompt is stored in `.ralph/prompt.txt` and is **re-read on every iteration**. This means you can edit the prompt file while Ralph is running, and your changes will take effect on the next iteration.

**Example:**
```bash
# Start ralph
ralph "Build a todo API" 20

# While ralph is running, edit the prompt to add more context
echo "Build a todo API with user authentication and rate limiting" > .ralph/prompt.txt

# Ralph will use the updated prompt on the next iteration
```

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

- `prompt.txt` - The current prompt (editable while running)
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

- **Start small**: The default is 100 iterations, but you can specify fewer (5-10) if you want to check progress more frequently
- **Check status**: Use `--status` to see progress before resuming
- **Review the log**: Check `.ralph/ralph.log` for detailed execution history
- **Edit tasks manually**: You can edit `.ralph/tasks.md` to add, remove, or reorder tasks
- **Update prompt dynamically**: Edit `.ralph/prompt.txt` while ralph runs to adjust instructions
- **Use pre-planned mode**: Use `--pre-planned` when you want to define the task list yourself

## License

MIT
