1. Think First (Plan Mode)

  How to activate:
  Shift + Tab + Tab    # Press Shift+Tab twice to enter plan mode

  Workflow:

1. Before typing any code request, enter plan mode
2. Describe what you want to build in detail
3. Ask Claude for architecture options
4. Discuss tradeoffs back and forth
5. Agree on approach, then exit plan mode and implement

---

2. CLAUDE.md File

  Create the file:

# In your project root

  touch CLAUDE.md

  Example CLAUDE.md structure:

# Project: MyApp

## Tech Stack

- TypeScript with strict mode (we've had production bugs from implicit any)
- React 18 with hooks
- PostgreSQL with Prisma ORM

## Commands

- `npm run dev` - Start development server
- `npm run test` - Run tests
- `npm run lint` - Lint and fix

## Conventions

- Use kebab-case for file names
- All API routes under /api/v1/
- Never use console.log in production code - use our logger utility

## Known Quirks

- The auth middleware expects JWT in Authorization header, not cookies
- Database migrations must be run manually: `npx prisma migrate dev`

  Update on the fly:

- Press # key while in Claude Code to add instructions automatically

---

3. Manage Context Windows

  Commands to use:
  /compact     # Summarize and compress current context
  /clear       # Clear context entirely and start fresh

  Create external memory files:
  touch SCRATCHPAD.md    # For notes and progress
  touch plan.md          # For detailed implementation plans

  Copy-paste reset workflow:

1. Copy important info from terminal
2. Run /compact
3. Run /clear
4. Paste back only what matters

---

4. Prompting Best Practices

  Bad prompt:
  Build me an auth system

  Good prompt:
  Build email/password authentication:

- Use the existing User model in src/models/user.ts
- Store sessions in Redis with 24-hour expiry
- Add middleware protecting routes under /api/protected
- Keep it simple - one file if possible
- Don't add abstractions I didn't ask for
- This runs on every request so it needs to be fast

---

5. Model Selection

  Switch models in Claude Code:
  Shift + Tab    # Toggle between Sonnet and Opus

  Recommended workflow:

1. Use Opus for: planning, architecture decisions, complex debugging
2. Switch to Sonnet for: implementation, boilerplate, refactoring

---

6. Tools & Configuration

  MCP Servers

  Configure in ~/.claude/settings.json:
  {
    "mcpServers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "your-token-here"
        }
      },
      "slack": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-slack"],
        "env": {
          "SLACK_TOKEN": "your-token-here"
        }
      }
    }
  }

  Hooks

  Configure in ~/.claude/settings.json:
  {
    "hooks": {
      "postToolUse": [
        {
          "matcher": "Edit|Write",
          "command": "npx prettier --write $FILE"
        },
        {
          "matcher": "Edit|Write",
          "command": "npx eslint --fix $FILE"
        }
      ]
    }
  }

  Custom Slash Commands

  Create command files:
  mkdir -p .claude/commands

  Example: .claude/commands/review.md
  Review the current file for:

- Security vulnerabilities
- Performance issues
- Code style violations
- Missing error handling

  Be concise. List issues as bullet points with line numbers.

  Use it:
  /review

---

7. When Claude Gets Stuck

  Recovery steps:
  /clear                    # Step 1: Fresh start

  Then try one of these approaches:

- Simplify: Break the task into smaller pieces
- Show example: Write a minimal working example yourself, ask Claude to follow the pattern
- Reframe: "Implement this as a state machine" instead of "handle these transitions"

---

8. Build Systems (Headless Mode)

  Run Claude non-interactively:

# Basic usage

  claude -p "Explain what this function does" < src/utils.ts

# With output to file

  claude -p "Review this PR for issues" > review.md

# Chain with other tools

  git diff HEAD~1 | claude -p "Summarize these changes for a commit message"

  Example: Automated PR Review Script
  #!/bin/bash

# .github/scripts/claude-review.sh

  PR_DIFF=$(gh pr diff $PR_NUMBER)
  claude -p "Review this PR diff for bugs, security issues, and style problems: $PR_DIFF" > review.md
  gh pr comment $PR_NUMBER --body-file review.md

  Example: Auto-update docs
  #!/bin/bash

# Update API docs when routes change

  claude -p "Generate OpenAPI documentation for these routes" < src/routes/*.ts > docs/api.yaml

---

  Quick Setup Checklist

# 1. Create CLAUDE.md in project root

  touch CLAUDE.md

# 2. Create scratch files

  touch SCRATCHPAD.md plan.md

# 3. Create custom commands folder

  mkdir -p .claude/commands

# 4. Configure settings (optional)

  code ~/.claude/settings.json
