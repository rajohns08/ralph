# Ralph

An automated builder/reviewer loop that uses Claude for implementation and Codex for code review.

## Overview

Ralph takes a user-provided task list and iteratively implements and reviews each task:

1. **Builder** (Claude) - Implements one task at a time
2. **Reviewer** (Codex) - Reviews the implementation and approves or rejects with feedback
3. If rejected, the builder addresses feedback and resubmits

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

### Setup

Before running ralph, create your task list and prompt:

```bash
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
```

### Run

```bash
ralph [prompt] [max_iterations]
```

**Examples:**
```bash
# Use existing .ralph/prompt.txt and .ralph/tasks.md
ralph

# Provide/update the prompt
ralph "Build a todo API with authentication"

# Set max iterations
ralph 10

# Provide prompt and max iterations
ralph "Build a todo API" 10
```

The `max_iterations` argument is optional and defaults to 100.

### Check status

```bash
ralph --status
```

Shows the current task list and progress summary.

### Dynamic Prompt Updates

The prompt is stored in `.ralph/prompt.txt` and is **re-read on every iteration**. This means you can edit the prompt file while Ralph is running, and your changes will take effect on the next iteration.

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
┌─────────────┐     ┌─────────────┐
│   Builder   │────>│  Reviewer   │
│  (Claude)   │<────│   (Codex)   │
└─────────────┘     └─────────────┘
    [ ] -> [R]         [R] -> [x] (approved)
    [!] -> [R]         [R] -> [!] (rejected)
```

### Files Created

Ralph uses a `.ralph/` directory containing:

- `prompt.txt` - The current prompt (editable while running)
- `tasks.md` - The task list with status markers
- `ralph.log` - Full execution log
- `state` - Internal state tracking

### Git Integration

If running in a git repository, Ralph automatically commits after each builder phase with the task description as the commit message.

## Tips

- **Start small**: Use 5-10 iterations if you want to check progress frequently
- **Check status**: Use `--status` to see progress
- **Review the log**: Check `.ralph/ralph.log` for detailed execution history
- **Edit tasks manually**: You can edit `.ralph/tasks.md` to add, remove, or reorder tasks
- **Update prompt dynamically**: Edit `.ralph/prompt.txt` while ralph runs to adjust instructions

## License

MIT
