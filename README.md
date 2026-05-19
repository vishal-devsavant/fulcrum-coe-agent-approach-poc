# Fulcrum CoE — AI OS Repo

> Source of truth for the Fulcrum Centre of Excellence AI automation platform.
> Governed n8n workflow creation, testing, and publishing via Claude.

---

## What This Repo Does

This repo powers **two surfaces** for the Fulcrum CoE:

| Surface | Who uses it | How |
|---------|-------------|-----|
| **Claude Code / Claude Desktop** | DevSavant internal, power users | Open repo in Claude Desktop → Claude reads `CLAUDE.md` + `.claude/agents/` |
| **Claude Teams Shared Project** | Fulcrum employees (all staff) | Admin syncs `skills/` and `CLAUDE.md` content into the `Fulcrum CoE` Teams Project |

---

## Repo Structure

```
fulcrum-coe/
├── CLAUDE.md                          # Orchestrator instructions (Claude Code reads this)
├── .mcp.json                          # MCP server config for Claude Code (local)
│
├── .claude/
│   └── agents/
│       ├── n8n-builder.md             # Sub-agent: creates/updates n8n workflows
│       ├── n8n-tester.md              # Sub-agent: tests and debugs workflows
│       └── github-publisher.md        # Sub-agent: raises GitHub PR for review
│
├── .github/
│   └── workflows/
│       └── publish-n8n-workflow.yml   # CI: auto-activates workflow in n8n on PR merge
│
├── skills/                            # Mirror of agents — for Teams Project admin sync
│   ├── n8n-workflow-builder/SKILL.md
│   ├── n8n-workflow-tester/SKILL.md
│   └── github-asset-publisher/SKILL.md
│
├── workflows/                         # Workflow JSONs land here via PR
│   └── .gitkeep
│
└── docs/
    ├── adr/
    │   └── ADR-001-claude-teams-shared-project.md
    └── setup.md
```

---

## The Three-Phase User Journey

```
User: "I want a workflow that sends a Slack message when a Google Sheet row is added"
        │
        ▼
  [Orchestrator — CLAUDE.md]
  Understands intent, asks clarifying questions
        │
        ▼
  [n8n-builder agent]
  Builds workflow JSON, validates, saves to n8n as draft
        │
        ▼
  [n8n-tester agent]
  Runs safe test with simulated data, reports results
        │
        ▼ (after user confirms test passed)
  [github-publisher agent]
  Exports JSON, creates branch, opens PR on devsavant/fulcrum-coe
        │
        ▼
  [GitHub Actions CI]
  Admin merges PR → workflow auto-activates in n8n
```

---

## How Claude Code Uses This Repo

1. Open this repo folder in **Claude Desktop** (or Cursor with Claude Code extension)
2. Claude automatically reads `CLAUDE.md` as its orchestrator instructions
3. Claude automatically loads `.claude/agents/*.md` as specialist sub-agents
4. Claude automatically connects to n8n and GitHub MCP via `.mcp.json`
5. Just chat naturally — no commands, no code

---

## How the Teams Project Uses This

The `skills/` folder contains the **same content** as `.claude/agents/` but formatted as Claude Teams Skills (SKILL.md with YAML frontmatter). After admin review:

1. Admin zips each folder under `skills/`
2. Uploads to Claude Teams admin → Organization Skills
3. Adds n8n and GitHub as Org Connectors
4. Pastes `CLAUDE.md` content into the `Fulcrum CoE` Project instructions
5. Shares the Project org-wide

Employees open Claude Desktop → select `Fulcrum CoE` → same journey, no repo clone needed.

---

## Local Setup (Claude Code / Developer)

### Prerequisites
- Claude Desktop with Claude Code extension, or Cursor + Claude Code
- Node.js (for GitHub MCP server)
- Access to `vishalmishra.app.n8n.cloud`
- GitHub PAT with `repo` scope for `devsavant/fulcrum-coe`

### Environment Variables
Create a `.env` file (never commit this):
```
N8N_MCP_TOKEN=<your n8n MCP token>
GITHUB_TOKEN=<your GitHub PAT>
```

Or set them in your shell before opening Claude Desktop.

### GitHub Actions Secrets (repo settings)
| Secret | Value |
|--------|-------|
| `N8N_API_KEY` | n8n API key for activating workflows |

---

## MCP Servers Used

| Server | Purpose | URL / Command |
|--------|---------|---------------|
| n8n | Create, validate, test workflows | `https://vishalmishra.app.n8n.cloud/mcp-server/http` |
| GitHub | Create branches, commit files, open PRs | `npx @modelcontextprotocol/server-github` |

---

## Sync to Teams Project — Admin Checklist

When updating agents, sync to the Teams Project:

- [ ] Copy updated `CLAUDE.md` orchestrator text → Project instructions
- [ ] Re-package `skills/n8n-workflow-builder/` → upload as Org Skill
- [ ] Re-package `skills/n8n-workflow-tester/` → upload as Org Skill  
- [ ] Re-package `skills/github-asset-publisher/` → upload as Org Skill
- [ ] Verify n8n Connector URL matches `.mcp.json`
- [ ] Verify GitHub Connector is active on the Project

---

## Related Docs

- [ADR-001: Claude Teams Shared Project for Governed n8n Workflow Creation](docs/adr/ADR-001-claude-teams-shared-project.md)
- [Setup Guide](docs/setup.md)

---

*DevSavant — Product Engineering | Fulcrum AI OS*
