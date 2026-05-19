# ADR-001: Claude Teams Shared Project for Governed n8n Workflow Creation

**Status:** Proposed

**Date:** 2026-05-18

**Deciders:** DevSavant AI OS Track (Vishal M), pending Fulcrum review (Jim Baker, Justin, Leonel)

**Tags:** architecture, ai-platform, claude-teams, n8n, integration, governance, non-technical-users

---

## Context

Fulcrum has standardized on **Claude Teams** as the organization-wide AI environment. Employees already use **Claude Code** on their machines. DevSavant is delivering the AI OS / CoE integration so Fulcrum staff—many of whom are **non-technical**—can create governed automation assets through natural language, without learning n8n, Git, or MCP concepts.

### Delivery model: shared Project, not shared repo

The **primary employee surface** is a **shared Claude Teams Project** (e.g. `Fulcrum CoE`) in claude.ai / Claude Desktop—not “everyone clones a CoE repo and opens Claude Code.”

| Surface | Who uses it | What “assets” are |
|---------|-------------|-------------------|
| **Shared Teams Project** | Fulcrum employees daily | Project **instructions**, **knowledge**, org **Connectors** (MCP), org **Skills** |
| **GitHub repo** | CI, governance, workflow JSON storage | Target for PRs via GitHub MCP; optional **source** for Skill `.md` files after admin sync |
| **CoE repo + Claude Code** | DevSavant / power users (internal) | `.claude/CLAUDE.md` + `.claude/agents/*.md` — **different product** (see [Platform comparison](#platform-comparison-and-key-conclusions)) |

Fulcrum’s **MCP credentials manager** is out of scope for this ADR—Fulcrum owns credential plumbing; DevSavant owns integration design and Skills/Connectors configuration.

### Target user journey

| Phase | User action | Expected system behavior |
|-------|-------------|---------------------------|
| **Create** | Describes a workflow in plain language inside the **shared Project** | Claude produces n8n workflow **JSON** via n8n MCP and Skills |
| **Test** | Validates the workflow (in n8n UI and/or via chat) | Workflow runs in n8n; user confirms behavior before promotion |
| **Publish** | Says *"publish the workflow"* or *"raise a PR"* (or uses n8n **Publish** when wired) | GitHub connector opens a PR containing workflow JSON; CI/governance gates run on the target repo |

### Architectural question

How should DevSavant deliver the **employee chat experience** for create → test → publish?

1. **Option 1 — Shared Claude Teams Project:** Admin provisions org-wide Connectors and Skills; members chat inside the Project.
2. **Option 2 — Non–Teams-native orchestration:** Explicit multi-agent routing via a **custom backend and UI**, or (related but **not** Option 1) the **Claude Code** repo pattern—documented below so “Method B” is not confused with the shared Project.

A POC on Claude Teams admin (shared Project + Connectors + Skills) is planned to validate Option 1 before full build-out.

---

## Decision Drivers

- **Non-technical UX:** Fulcrum employees must use the **shared Teams Project** and familiar chat—not terminals, repo clones, or a new portal unless unavoidable.
- **Time to value:** Client expects a working path soon; long greenfield backend work is a risk.
- **Governance:** Auditability, org-wide provisioning, and controlled tool access (n8n → test → GitHub PR).
- **Operational ownership:** Minimize DevSavant-run infrastructure Fulcrum must operate after handoff.
- **Platform accuracy:** Do not promise Claude Code subagent behaviour inside a Teams Project without Skills/Connectors mapping.
- **Future flexibility:** If the POC fails, escalate via TRD or a follow-up ADR (custom MCP, deterministic CI)—without blocking near-term Option 1.

---

## Considered Options

*Summaries below. Side-by-side comparison (Sarah’s journey, file mapping, and conclusions) is in [Platform comparison and key conclusions](#platform-comparison-and-key-conclusions)—read that section before [Decision Outcome](#decision-outcome).*

### Option 1: Shared Claude Teams Project + Org Connectors & Skills

**Description:** Fulcrum (or DevSavant as Owner) creates a shared Project `Fulcrum CoE` with custom **instructions** (orchestrator behaviour), optional **knowledge**, and org-provisioned **Connectors** (n8n MCP, GitHub MCP) plus **Skills** (`n8n-workflow-builder`, `github-asset-publisher`, `governance-check`). Members select the Project and chat; Claude uses implicit Skill routing and MCP tools in a **single conversation context**.

**Example flow:** *"Email my team when a Salesforce lead closes"* → relevant Skill loads → n8n MCP builds JSON → user tests in n8n → *"Raise a PR"* → GitHub MCP opens PR with workflow JSON.

See [Platform comparison and key conclusions](#platform-comparison-and-key-conclusions) for how this differs from a Claude Code repo with `.claude/agents/`.

**Pros:**
- Matches Fulcrum’s **shared Project** mandate; no employee repo clone.
- Native Teams governance: org Skills/Connectors, project sharing, audit on Team/Enterprise plans.
- **Lowest friction for non-tech users**—same UX as everyday Claude chat.
- **Days to first demo** after admin POC.
- Skills can be authored in Git and provisioned to the org after review.
- Aligns with Anthropic’s documented Teams sharing model.

**Cons:**
- **Not** Claude Code subagents: no `.claude/agents/` in Project; no per-agent isolated context or `Agent` tool in chat.
- All Project Connectors may be visible in one turn—higher wrong-tool risk than explicit sub-agents.
- Skill routing is implicit; harder to debug than orchestrator logs.
- Shared Skills are **view-only** for recipients; updates require admin re-provision or member re-install from directory ([Use Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)).

**Cost impact:** Claude Teams per-seat licensing; no separate orchestration API bill on the employee path.

**Risk level:** Low–Medium

---

### Option 2: Non–Teams-native orchestration (Method B)

Discovery referred to “orchestrator + sub-agents (n8n, GitHub, …).” That phrase maps to **different implementations**. Only **Option 2A** is the true alternative to Option 1 for the **employee** journey. **Option 2B** is documented so Roberto’s Claude Code repo pattern is not mistaken for Teams or for 2A.

#### Option 2A: Custom backend + explicit orchestrator (dedicated employee UI)

**Description:** Build orchestrator and specialized sub-agents in application code (Anthropic Messages API and/or **Managed Agents** beta), host a **custom web app** (e.g. GCP Cloud Run), authenticate users with **Fulcrum SSO (Okta)**. Each sub-agent runs with its own system prompt and tool allowlist (n8n MCP only, GitHub MCP only, etc.).

**Example flow:** User opens `coe.fulcrum.internal/chat` → Okta SSO → backend calls orchestrator → `n8n-builder` sub-agent → `github-publisher` sub-agent → PR created. User does **not** use the Teams shared Project for this flow.

Required only if Fulcrum needs **explicit** sub-agent isolation on the employee path and the shared Project POC is insufficient—see [Key conclusions](#key-conclusions-drive-the-decision-below) (items 4–5).

**Pros:**
- Strong tool isolation and explicit, traceable routing.
- Per-agent model selection (e.g. Haiku vs Opus).
- Deterministic governance steps in code.

**Cons:**
- **High build effort:** UI, backend, SSO, secrets, observability, RBAC—reimplements much of Teams.
- **Weeks to first demo**; conflicts with “ready soon.”
- New URL and UX—not the shared **Fulcrum CoE** Project.
- Managed Agents / multi-agent APIs may still be **beta** for production commitments.
- Higher per-request token cost (orchestrator + sub-agents per user message).
- Does **not** use admin-provisioned Teams Skills/Connectors for the chat surface.

**Cost impact:** Teams seats (if still used elsewhere) + API usage + Cloud Run/ops.

**Risk level:** High for V1

#### Option 2B: Claude Code repo with `.claude/agents/` (implicit orchestration — not selected for employees)

**Description:** Admin maintains a **GitHub repo** with `CLAUDE.md` and `.claude/agents/n8n-builder.md`, `github-publisher.md`, `governance-reviewer.md`. Anyone who opens the repo in **Claude Code** gets orchestrator + sub-agent delegation—**no custom backend, no custom UI** ([Claude Code subagents](https://code.claude.com/docs/en/sub-agents)).

This is Roberto’s **repo + `.claude/agents/`** pattern—the **left column** in the [platform comparison](#platform-comparison-and-key-conclusions) tables. Full journey and file mapping are documented there, not repeated here.

**Why this is not chosen for Fulcrum employees (V1):**
- Employee mandate is **Project-first** chat, not “open CoE repo in Claude Code.”
- Claude Code is terminal/IDE-oriented; poorer fit for broad non-tech staff than claude.ai Project chat.
- Org-wide distribution is **git access + local MCP setup**, not Owner provisioning in Teams admin.
- Subagent definitions in `.claude/` are **not** loaded when only adding directories via Claude Code `--add-dir` for files—subagents stay repo-bound ([Claude Code skills — configuration scope](https://docs.anthropic.com/en/docs/claude-code/skills)).

**Appropriate use:** DevSavant internal POC, authoring harness, and **translating** agent prompts into Option 1 Skills and Project instructions—not the Fulcrum employee CoE surface.

**Cost impact:** Claude Code / API on developer machines only.

**Risk level:** Low internally; **not applicable** as primary employee delivery

#### Option 2 comparison summary

| Criterion | 2A Custom backend | 2B Claude Code repo | Option 1 Teams Project |
|-----------|-------------------|---------------------|-------------------------|
| Employee opens shared **Project** | No (custom URL) | No (opens repo in Code) | **Yes** |
| `.claude/agents/*.md` works as-is | N/A (code-defined agents) | **Yes** | **No** — use Skills |
| Custom backend required | **Yes** | **No** | **No** |
| Explicit sub-agent isolation | **Yes** | **Yes** | No (implicit Skills) |
| Fits “ready soon” | No | Fast for devs only | **Yes** |

---

## Platform comparison and key conclusions

*Anthropic product documentation review, May 2026. Read this section after [Considered Options](#considered-options) to see why Option 1 fits Fulcrum’s **shared Project** mandate and how it relates to Roberto’s **Claude Code repo** idea.*

Roberto’s design often looks like a **GitHub repo** with orchestrator + specialist `.md` files under `.claude/agents/`. Fulcrum’s requirement is a **shared Teams Project** where employees chat without cloning that repo. Both approaches can produce n8n workflow JSON and a GitHub PR—the difference is **where Sarah works** and **how agents are represented**.

**Scenario:** Sarah (operations) asks: *“When a Salesforce lead closes, email my team.”* She tests in n8n, then promotes the workflow via GitHub PR.

### Table 1 — Sarah’s journey: Claude Code repo vs shared Teams Project

| Step | Claude Code + CoE repo | Shared Project **Fulcrum CoE** |
|------|------------------------|--------------------------------|
| **1. Where Sarah works** | Clones repo; opens folder in **Claude Code** (terminal/IDE) | Opens **Claude Desktop** → selects **Fulcrum CoE** in sidebar (**no git clone**) |
| **2. Orchestrator (“who coordinates”)** | Reads `CLAUDE.md` in the repo | Reads **Project custom instructions** (admin pasted same intent as `CLAUDE.md`) |
| **3. Build workflow JSON** | Main agent **delegates** to `n8n-builder` **sub-agent** (new context; n8n tools only) → n8n MCP | **One chat**: Claude loads **`n8n-workflow-builder` Skill** → uses **n8n Connector** |
| **4. How routing happens** | **Explicit** — `Agent` tool spawns sub-agent ([Claude Code subagents](https://code.claude.com/docs/en/sub-agents)) | **Implicit** — Claude picks Skill from description; **no** separate sub-agent session |
| **5. Test** | Sarah tests workflow in n8n | Same — Sarah tests in n8n |
| **6. Publish (PR)** | Main agent **delegates** to `github-publisher` **sub-agent** (GitHub tools only) → PR | Sarah says *“raise a PR”* → **`github-asset-publisher` Skill** + **GitHub Connector** → PR |
| **7. Tool access model** | Each sub-agent sees **only** its MCP tools | **All** Project Connectors may be visible in the same chat (weaker isolation) |
| **8. What Fulcrum admin set up** | Repo on GitHub; developers need clone + local MCP config | Owner provisioned **Skills + Connectors** once; Project shared org-wide |

*Maps to options: **left column** = Option 2B (Claude Code repo). **Right column** = Option 1 (shared Project — recommended for employees).*

### Table 2 — What you configure: repo files vs Teams Project

| Concept | In Claude Code repo | In shared Teams Project | Practical difference |
|---------|---------------------|-------------------------|----------------------|
| Orchestrator rules | `CLAUDE.md` | **Project instructions** | Same words possible; different place to paste them |
| n8n specialist | `.claude/agents/n8n-builder.md` | Org **Skill** `n8n-workflow-builder` (`SKILL.md` in ZIP) | File in repo vs Skill uploaded in admin |
| GitHub specialist | `.claude/agents/github-publisher.md` | Org **Skill** `github-asset-publisher` | Same roles; Teams uses Skills, not `agents/` folder |
| Tool wiring | `.mcp.json` (local) | **Connectors** (n8n + GitHub MCP on Project) | Per-machine config vs org-wide enablement |
| How staff get access | Clone repo + open in Code | Open shared **Fulcrum CoE** Project | Repo path = developers; Project path = employees |

### Repo layout (Claude Code only — not loaded by Teams chat)

```text
Fulcrum CoE Repo (GitHub)          ← optional source of truth; NOT the employee chat surface
└── .claude/
    ├── CLAUDE.md                  ← orchestrator
    └── agents/
        ├── n8n-builder.md
        ├── github-publisher.md
        └── governance-reviewer.md
```

### Key conclusions (drive the decision below)

1. **You cannot mount `.claude/agents/` in a shared Project** and get Claude Code–style sub-agents. Teams supports **instructions + Skills + Connectors** only ([What are projects?](https://support.claude.com/en/articles/9517075-what-are-projects), [Use Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)).
2. **Fulcrum employee path = right column (Table 1)** — shared Project chat; admin provisions Skills and Connectors org-wide ([Provision and manage Skills](https://support.claude.com/en/articles/13119606-provisioning-and-managing-skills-for-your-organization), [MCP connectors](https://support.claude.com/en/articles/14503689-mcp-connectors), [Project visibility and sharing](https://support.claude.com/en/articles/9519189-project-visibility-and-sharing)).
3. **Roberto’s repo pattern = left column (Table 1)** — valid in **Claude Code** with **no custom backend** (Option 2B); use internally to prototype, then **translate** content into Skills and Project instructions for Option 1.
4. **Implicit orchestration without a backend** means the Claude Code repo—not the Teams Project. **Explicit** sub-agents for all employees on day one means Option 2A (custom portal) or future Anthropic product; there is no org-shared “Agent” folder in Projects today ([Introducing Agent Skills](https://www.anthropic.com/news/skills)).
5. **Git sync (recommended):** Keep Skill text and instructions in Git for review; after approval, admin syncs to Teams. Employees only open the **shared Project**.

---

## Decision Outcome

**Chosen option:** **Option 1** — Shared Claude Teams Project + org Connectors and Skills (pending POC validation; status remains **Proposed** until admin POC completes).

**Rationale:**

1. **Shared Project is the requirement** — Fulcrum’s employee experience is org-shared Project chat, not a shared CoE repo (see [Delivery model](#delivery-model-shared-project-not-shared-repo)).
2. **Platform reality** — Conclusions 1–2 in [Platform comparison](#platform-comparison-and-key-conclusions): `.claude/agents/` does not work inside a Project; Table 1 **right column** is the employee path.
3. **Non-technical UX and time to value** — Option 1 delivers create → test → PR in days; Option 2A forces a new portal; Option 2B forces Claude Code (Table 1 **left column**).
4. **Governance** — Team/Enterprise org provisioning and project sharing are documented admin capabilities; Option 2A rebuilds RBAC.
5. **Roberto’s repo pattern** — Conclusion 3: valid as Git source of truth and Claude Code POC (Option 2B); **translate** into Option 1 Skills and instructions—not mount the repo in Teams.

Deterministic **GitHub Actions** on PR open complement Option 1; they are not a reason to adopt Option 2A for employee chat.

**Consequences:**
- Fast POC on Teams admin; low ops burden; no employee repo clone.
- We accept implicit Skill routing; mitigated with few Skills, tight descriptions, and chat confirmation before PR.
- DevSavant maintains parallel Claude Code agent files optionally to prototype flows before Skills sync.
- PR identity and MCP credential pass-through must be validated in POC (TRD).
- If POC fails routing or compliance, escalate to hosted custom MCP or ADR-002—not default to Option 2A portal.

---

## Implementation Notes

High-level delivery only; configuration detail belongs in the TRD.

| Layer | Approach |
|-------|----------|
| Employee surface | Shared Teams Project `Fulcrum CoE` (not shared repo for chat) |
| Orchestrator | Project **custom instructions** (from `CLAUDE.md` content, synced by admin) |
| Specialist behaviours | Org **Skills**: `n8n-workflow-builder`, `github-asset-publisher`, `governance-check` |
| Tools | Org **Connectors**: n8n MCP, GitHub MCP |
| Git | Workflow JSON via GitHub MCP → PR; Skills/instructions optionally versioned in Git then provisioned |
| POC gate | Teams admin POC before status → **Accepted** |

**Publish / PR triggers (V1 vs later):**

| Trigger | V1 handling |
|---------|-------------|
| Chat: *"raise a PR"* / *"publish to GitHub"* | `github-asset-publisher` Skill + GitHub MCP |
| User confirms testing in chat | Skill blocks GitHub MCP until confirmation |
| n8n **Publish** button → auto PR | Deferred to TRD |

**TRD reference:** To be created — n8n MCP, GitHub layout, PR template, CI gates, Skill sync procedure, Fulcrum MCP credentials interface.

**Out of scope for V1:** Option 2A custom portal; employee-facing Option 2B (Claude Code as CoE UI).

---

## Related ADRs

- None (first architecture decision for Fulcrum AI OS integration). Escalation patterns (custom MCP) may become ADR-002 if POC requires.

---

## Notes

- **POC gate:** Move to **Accepted** only after shared Project member walkthrough: n8n JSON → test → PR.
- **Claude Code:** Continue Option 2B POC internally; map learnings to Skill `SKILL.md` files for Option 1.
- **Open TRD items:** Production n8n on GCP; GitHub commit identity; n8n Publish webhook; credentials manager interface.

**Follow-up (optional):** OpenSpec for author repos (engineering harness)—see research workplan; ADR-002 if Fulcrum wants a formal spec-tool decision.

---

## References (official)

| Topic | Source |
|-------|--------|
| Projects (instructions, knowledge, sharing) | [What are projects?](https://support.claude.com/en/articles/9517075-what-are-projects) |
| Project sharing (Team / Enterprise) | [Project visibility and sharing](https://support.claude.com/en/articles/9519189-project-visibility-and-sharing) |
| Skills in claude.ai | [Use Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude) |
| Org-wide Skill provisioning | [Provision and manage Skills for your organization](https://support.claude.com/en/articles/13119606-provisioning-and-managing-skills-for-your-organization) |
| Agent Skills (product overview) | [Introducing Agent Skills](https://www.anthropic.com/news/skills) |
| MCP Connectors (org admin) | [MCP connectors](https://support.claude.com/en/articles/14503689-mcp-connectors) |
| Pre-built / remote MCP | [Use connectors to extend Claude](https://support.claude.com/en/articles/11176164-pre-built-connectors-using-remote-mcp) |
| Claude Code subagents (`.claude/agents/`) | [Create custom subagents](https://code.claude.com/docs/en/sub-agents) |
| Claude Code Skills vs other `.claude/` config | [Extend Claude with skills](https://docs.anthropic.com/en/docs/claude-code/skills) |
| Skills directory (Team plan) | [Browse skills, connectors, and plugins](https://support.claude.com/en/articles/14328846-browse-skills-connectors-and-plugins-in-one-directory) |

---

## Architecture Decision Records Index

| ADR | Title | Status | Date | Tags |
|-----|-------|--------|------|------|
| ADR-001 | Claude Teams Shared Project for Governed n8n Workflow Creation | Proposed | 2026-05-18 | architecture, ai-platform, claude-teams, n8n, integration, governance |

## By Category

### AI Platform / Integration
- ADR-001: Claude Teams Shared Project for Governed n8n Workflow Creation

---

*DevSavant — Product Engineering*
