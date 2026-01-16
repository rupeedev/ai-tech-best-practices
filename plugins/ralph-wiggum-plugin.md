# Ralph Loop Plugin for Claude Code

Implementation of the Ralph Wiggum technique for iterative, self-referential AI development loops.

## What is Ralph Loop?

Ralph Loop is a development methodology based on continuous AI agent loops. As Geoffrey Huntley describes it: **"Ralph is a Bash loop"** - a simple `while true` that repeatedly feeds an AI agent a prompt, allowing it to iteratively improve its work until completion.

### How It Works

1. You provide a task with clear success criteria
2. Claude works on the task
3. When Claude tries to exit, a Stop hook blocks it
4. The same prompt is fed back in
5. Repeat until completion or max iterations reached

The key insight: **the prompt stays the same, but the codebase changes**. Each iteration sees the modified files and can build on previous work.

---

## Installation

### Prerequisites

- Claude Code CLI installed
- Access to the official plugins marketplace

### Step 1: Ensure Marketplace is Registered

Check if the official marketplace is already set up:

```bash
cat ~/.claude/plugins/known_marketplaces.json
```

If not present, the marketplace will be auto-registered when you first use plugins.

### Step 2: Update the Marketplace (Pull Latest)

```bash
cd ~/.claude/plugins/marketplaces/claude-plugins-official
git pull origin main
```

### Step 3: Get the Current Git SHA

```bash
cd ~/.claude/plugins/marketplaces/claude-plugins-official
git rev-parse HEAD
# Example output: 96276205880a60fd66bbae981f5ab568e70c4cbf
```

### Step 4: Create Plugin Cache Directory

```bash
mkdir -p ~/.claude/plugins/cache/claude-plugins-official/ralph-loop/<GIT_SHA>
```

Replace `<GIT_SHA>` with the actual SHA from Step 3.

### Step 5: Copy Plugin Files to Cache

```bash
cp -r ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/ralph-loop/* \
      ~/.claude/plugins/cache/claude-plugins-official/ralph-loop/<GIT_SHA>/

cp -r ~/.claude/plugins/marketplaces/claude-plugins-official/plugins/ralph-loop/.claude-plugin \
      ~/.claude/plugins/cache/claude-plugins-official/ralph-loop/<GIT_SHA>/
```

### Step 6: Enable Plugin in Settings

Edit `~/.claude/settings.json` and add to `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "ralph-loop@claude-plugins-official": true
  }
}
```

### Step 7: Register in Installed Plugins

Edit `~/.claude/plugins/installed_plugins.json` and add:

```json
{
  "ralph-loop@claude-plugins-official": [
    {
      "scope": "user",
      "installPath": "/Users/<USERNAME>/.claude/plugins/cache/claude-plugins-official/ralph-loop/<GIT_SHA>",
      "version": "<GIT_SHA>",
      "installedAt": "2026-01-16T00:00:00.000Z",
      "lastUpdated": "2026-01-16T00:00:00.000Z",
      "gitCommitSha": "<GIT_SHA_FULL>"
    }
  ]
}
```

Replace:
- `<USERNAME>` with your actual username
- `<GIT_SHA>` with the short SHA (first 12 chars)
- `<GIT_SHA_FULL>` with the full 40-character SHA

### Step 8: Restart Claude Code

```bash
# Exit current session and start fresh
claude
```

### Verification

After restart, verify the plugin is loaded:

```bash
/help
```

You should see `/ralph-loop` and `/cancel-ralph` commands available.

---

## Quick Reference

| Location | Path |
|----------|------|
| Plugin cache | `~/.claude/plugins/cache/claude-plugins-official/ralph-loop/` |
| Settings | `~/.claude/settings.json` |
| Installed plugins | `~/.claude/plugins/installed_plugins.json` |

---

## How to Use

### Start a Ralph Loop

```bash
/ralph-loop "<prompt>" --max-iterations <n> --completion-promise "<text>"
```

**Options:**
- `--max-iterations <n>` - Stop after N iterations (default: unlimited)
- `--completion-promise <text>` - Exact phrase that signals completion

### Cancel a Running Loop

```bash
/cancel-ralph
```

---

## Example Usage

### Test Coverage Sprint

```bash
/ralph-loop "Add tests for all uncovered functions in src/utils.
Success criteria:
- Coverage >= 85%
- All tests green
- No lint errors
Output <promise>COMPLETE</promise> when done." \
--max-iterations 20 \
--completion-promise "COMPLETE"
```

### Build a Simple API

```bash
/ralph-loop "Build a minimal TODO API.
Requirements:
- CRUD endpoints
- Input validation
- Tests for each endpoint
Output <promise>READY</promise> when done." \
--max-iterations 30 \
--completion-promise "READY"
```

### Legacy Migration

```bash
/ralph-loop "Migrate tests from Jest to Vitest.
Success criteria:
- All tests pass
- Snapshots unchanged
- Updated config files
- README updated
Output <promise>DONE</promise> when complete." \
--max-iterations 25 \
--completion-promise "DONE"
```

### Codebase Standardization

```bash
/ralph-loop "Standardise error handling in src/:
- Replace inline string errors with Error subclasses
- Add error tests where missing
- Keep public API unchanged
Output <promise>STANDARDISED</promise> when done." \
--max-iterations 15 \
--completion-promise "STANDARDISED"
```

---

## Prompt Template

```markdown
TASK:
<one sentence describing the change>

SUCCESS CRITERIA:
- <measurable requirement 1>
- <measurable requirement 2>
- <test/build/linters must pass>

PROCESS:
1) Make the smallest change that moves toward success
2) Run tests or validation
3) Fix failures and repeat
4) If stuck after N iterations, summarise blockers and suggest next steps

OUTPUT:
<promise>YOUR_PROMISE</promise> only when ALL criteria are met
```

---

## References

- [Original technique by Geoffrey Huntley](https://ghuntley.com/ralph/)
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)
- [Paddo.dev: Ralph Wiggum Autonomous Loops](https://paddo.dev/blog/ralph-wiggum-autonomous-loops/)
- [HumanLayer: Brief History of Ralph](https://www.humanlayer.dev/blog/brief-history-of-ralph)
