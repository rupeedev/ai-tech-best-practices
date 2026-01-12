# AI & Tech Best Practices

A curated collection of knowledge base articles, tutorials, and best practices for modern AI-assisted development, MCP (Model Context Protocol) integrations, and full-stack development patterns.

## What's Inside

### MCP Server Integrations (`KBs/mcp/`)
Step-by-step guides for connecting Claude Code to various services:

| Service | Description |
|---------|-------------|
| **Notion** | Read/write Notion pages and databases |
| **Linear** | Project management and issue tracking |
| **Atlassian** | Jira and Confluence integration |
| **Sentry** | Error monitoring and debugging |
| **Stripe** | Payment processing queries |
| **Trivago** | Hotel and accommodation search |
| **Hugging Face** | ML models, datasets, and papers |
| **PostgreSQL** | Database querying via DBHub |

### AI & LLM Resources (`KBs/`)
- **Claude Code Setup** - Best practices for Claude Code CLI
- **Git Worktrees** - Parallel agent development strategies
- **Self-Hosted LLM** - Deploy local AI with Ollama + Open WebUI
- **MCP RAG** - Build RAG systems with 200+ data sources
- **Agent Gateway** - Security for AI agent communications
- **Master Prompts** - Effective prompt engineering patterns

### Dashboard & BI Tools (`KBs/`)
Comparisons and setup guides for:
- NocoBase, Metabase, Grafana, Redash, Appsmith, Wren AI

### Temporal Workflows (`KBs/temporal/`)
Documentation and presentations on Temporal workflow orchestration for reliable AI agents.

### Full-Stack Development (`docs/`)
- Authentication & Authorization patterns
- Security best practices (OWASP Top 10)
- Data fetching & mutation patterns
- API endpoint design
- React Server Components
- Routing strategies

### MCP Tools & Utilities (`mcp/`)

| Tool | Description |
|------|-------------|
| `vulnerability_scanner.py` | Scan codebase for exposed secrets, API keys, and credentials |
| `claude_setup.py` | Automated Claude Code project setup (CLAUDE.md, commands, hooks) |
| `db_server.py` | Database MCP server for Supabase/Postgres operations |
| `ikanban.py` | iKanban CLI & MCP server for task management |
| `server.py` | Vibe Kanban MCP server for AI coding agents |
| `cli.py` | vibe-kanban CLI client for direct API access |
| `deploy.py` | Railway deployment script for frontend/backend |
| `setup_statusline.py` | Configure Claude Code status line (model, tokens, cost) |
| `MCP-Setup-TUTORIAL.md` | Step-by-step MCP server setup guide |

## How You Can Benefit

**For AI/LLM Developers:**
- Ready-to-use MCP server configurations
- Best practices for Claude Code workflows
- Patterns for building reliable AI agents with Temporal

**For Full-Stack Developers:**
- Security-first development guidelines
- Modern data fetching patterns
- API design references

**For DevOps/Platform Engineers:**
- Self-hosted AI infrastructure guides
- Dashboard tool comparisons
- Git branching strategies

## Quick Start

1. Browse the `KBs/` folder for AI and tool integrations
2. Check `docs/` for development best practices
3. Use `templates/` for project scaffolding

## Vulnerability Scanner

Before sharing your code publicly, scan for exposed secrets using the included vulnerability scanner.

### Features

Detects **30+ secret patterns** including:
- API keys (AWS, OpenAI, Anthropic, GitHub, Slack, Stripe, Google, etc.)
- Database connection strings with credentials
- Private keys (RSA, SSH, PGP, EC)
- Bearer tokens & JWTs
- Passwords in config files
- Cloud credentials (Azure, GCP)

### Usage

```bash
# Scan current directory
python3 mcp/vulnerability_scanner.py .

# Scan specific path
python3 mcp/vulnerability_scanner.py /path/to/project

# Verbose output (show files being scanned)
python3 mcp/vulnerability_scanner.py . -v

# Export findings to JSON
python3 mcp/vulnerability_scanner.py . -o report.json
```

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | No issues found |
| `1` | HIGH severity findings |
| `2` | CRITICAL severity findings |

### Example Output

```
================================================================================
VULNERABILITY SCAN REPORT
================================================================================

Root Path: /your/project
Files Scanned: 150

SUMMARY
  CRITICAL: 0
  HIGH: 0
  MEDIUM: 0
  LOW: 0

No secrets or vulnerabilities detected!
================================================================================
```

### CI/CD Integration

Add to your GitHub Actions workflow:

```yaml
- name: Security Scan
  run: python3 mcp/vulnerability_scanner.py . && echo "No secrets found!"
```

## Credits & Attributions

This repository curates and builds upon excellent work from the community. Full credit to the original authors:

### Medium Authors

| Author | Article | Link |
|--------|---------|------|
| **Kenji Onisuka** | Claude Code and Git Worktrees: Scaling Agentic Coding | [medium.com/@kenji-onisuka](https://medium.com/@kenji-onisuka) |
| **Civil Learning** | Building a Local MCP-Powered RAG with 200+ Data Sources | [medium.com/@civillearning](https://medium.com/@civillearning) |
| **Vijay Gadhave** | How I Built a Self-Hosted AI Server in 5 Minutes | [medium.com/@vijaygadhave2014](https://medium.com/@vijaygadhave2014) |
| **Deep Concept** | The Only Master Prompt for ChatGPT | [medium.com/@Deep-concept](https://medium.com/@Deep-concept) |
| **NocoBase** | 6 Best Open-Source AI Tools to Build Dashboards | [nocobase.com/blog](https://www.nocobase.com/en/blog/6-best-open-source-ai-tools-to-build-dashboards) |
| **Algo Insights** | MCP for Developers Guide | [medium.com/@algoinsights](https://medium.com/@algoinsights) |

### Official Documentation & Projects

- [Anthropic Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Agent Gateway](https://github.com/agentgateway/agentgateway)
- [Graphiti MCP Server](https://github.com/getzep/graphiti)
- [Opik MCP Server](https://github.com/comet-ml/opik-mcp)
- [MindsDB](https://docs.mindsdb.com/model-context-protocol)
- [Obsidian Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api)

### Open Source Tools Referenced

- [Temporal](https://temporal.io/) - Workflow orchestration
- [Metabase](https://www.metabase.com/) - Business intelligence
- [Grafana](https://grafana.com/) - Observability platform
- [Redash](https://redash.io/) - Data visualization
- [NocoBase](https://www.nocobase.com/) - No-code platform
- [Wren AI](https://www.getwren.ai/) - AI-powered BI
- [Appsmith](https://www.appsmith.com/) - Internal tools builder

## Structure

```
.
├── KBs/                         # Knowledge base articles
│   ├── mcp/                     # MCP server integration guides
│   ├── temporal/                # Temporal workflow docs
│   └── Tools-Setup/             # Tool configuration guides
├── docs/                        # Development documentation
├── mcp/                         # MCP tools & utilities
│   ├── vulnerability_scanner.py # Secret detection tool
│   ├── claude_setup.py          # Project setup automation
│   ├── db_server.py             # Database MCP server
│   ├── ikanban.py               # Task management CLI/MCP
│   ├── server.py                # Vibe Kanban MCP server
│   ├── cli.py                   # CLI client
│   ├── deploy.py                # Deployment script
│   └── setup_statusline.py      # Status line config
├── templates/                   # Project templates
└── tracking/                    # Task and changelog tracking
```

## Contributing

Found an error or want to add more resources? Contributions are welcome!

## License

This is a collection of curated resources. Please respect the original authors' licenses and attributions when using this material.

---

*Built with the help of [Claude Code](https://claude.ai/claude-code)*
