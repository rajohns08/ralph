# Ralph

An automated builder/reviewer loop that uses configurable AI tools for implementation and code review.

## Overview

Ralph takes a user-provided task list and iteratively implements and reviews each task:

1. **Builder** (default: Claude) - Implements one task at a time
2. **Reviewer** (default: Codex) - Reviews the implementation and approves or rejects with feedback
3. If rejected, the builder addresses feedback and resubmits

This continues until all tasks are complete or the maximum iterations are reached.

## Requirements

- Bash
- Git (optional, for automatic commits)
- At least one supported AI tool:
  - [Claude CLI](https://github.com/anthropics/claude-code) (`claude`) - default builder
  - [Codex CLI](https://github.com/openai/codex) (`codex`) - default reviewer
  - [Gemini CLI](https://github.com/google-gemini/gemini-cli) (`gemini`)
  - [GitHub Copilot CLI](https://github.com/github/gh-copilot) (`copilot`)

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
ralph [options] [prompt] [max_iterations]
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

# Use Gemini as builder and Copilot as reviewer
ralph --builder=gemini --reviewer=copilot "Build a todo API"
```

The `max_iterations` argument is optional and defaults to 100.

### Configuring tools

By default, Ralph uses Claude as the builder and Codex as the reviewer. You can change this with CLI flags or config files.

**CLI flags** (highest priority):
```bash
ralph --builder=gemini --reviewer=claude
ralph --collab --builder=copilot --reviewer=gemini "topic"

# Extra arguments are passed to the tool command after its system flags:
ralph --builder="copilot --model gpt-5.2-codex" --reviewer="copilot --model gpt-5.2"
```

**Global config** (`~/.ralph/config`) — defaults for all projects:
```
builder=copilot --model gpt-5.2-codex
reviewer=copilot --model gpt-5.2
```

**Project config** (`.ralph/config`) — overrides global per-key:
```
builder=claude
reviewer=codex
```

Extra arguments after the tool name are passed to the command.

**Precedence** (later overrides earlier, per-key):
1. Defaults (builder=claude, reviewer=codex)
2. `~/.ralph/config` (global)
3. `.ralph/config` (project)
4. CLI flags (`--builder`, `--reviewer`)

Supported tools: `claude`, `codex`, `gemini`, `copilot`

Each tool is launched with appropriate flags:
| Tool | Command |
|------|---------|
| Claude | `claude --dangerously-skip-permissions --verbose --print --output-format stream-json --include-partial-messages` |
| Codex | `codex exec --yolo --skip-git-repo-check --json` |
| Gemini | `gemini --yolo` |
| Copilot | `copilot --allow-all-tools` |

The builder tool is also used for collab turn 1 (odd turns) and collab summary generation. The reviewer tool is used for collab turn 2 (even turns).

### Check status

```bash
ralph --status
```

Shows the current task list and progress summary.

### Collab mode

```bash
ralph --collab "topic to discuss"
```

Starts a collaborative discussion between the builder and reviewer tools on the given topic. They take turns until both agree, then a summary is generated and saved to `.ralph/collab.md`.

### Fix issue mode

```bash
ralph --fix-issue "description of the issue"
```

Combines collab and the standard builder/reviewer loop:

1. **Phase 1 (Collab):** Builder and reviewer tools discuss the issue and agree on an approach
2. **Phase 2 (Build):** Automatically creates a task from the collab summary and runs the builder/reviewer loop to implement the fix

This is useful when you want the two models to discuss a problem before implementing a solution, without running `--collab` and standard mode separately.

### Deep review mode

```bash
ralph --deep-review "review feature-branch relative to main"
```

A three-phase code review process:

1. **Phase 1 (Collab):** Builder and reviewer tools discuss the changes and agree on a review checklist, sized so no single review overloads context (one item for small changes, per-file for large ones)
2. **Phase 2 (Review):** Each checklist item is reviewed independently by both tools — the builder tool as Reviewer 1 and the reviewer tool as Reviewer 2. Each writes notes and flags any blocking issues
3. **Phase 3 (Summary):** Findings are consolidated into a final summary with an approve/reject recommendation

Output is saved to `.ralph/deep-review.md` with the final summary also printed to the console.

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
│ (configurable) │<────│ (configurable) │
└─────────────┘     └─────────────┘
    [ ] -> [R]         [R] -> [x] (approved)
    [!] -> [R]         [R] -> [!] (rejected)
```

### Files

**Global** (`~/.ralph/`):
- `config` - Global tool defaults (apply to all projects)

**Project** (`.ralph/`):
- `config` - Project-specific tool configuration (overrides global)
- `prompt.txt` - The current prompt (editable while running)
- `tasks.md` - The task list with status markers
- `collab.md` - Collab discussion transcript and summary (created by `--collab` and `--fix-issue`; also used for the planning discussion in `--deep-review`)
- `deep-review.md` - Deep review checklist, reviewer notes, and final summary (created by `--deep-review`)
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
