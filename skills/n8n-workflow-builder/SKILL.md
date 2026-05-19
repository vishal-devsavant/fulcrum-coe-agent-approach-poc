---
name: n8n-workflow-builder
description: >
  Use this skill whenever a team member wants to CREATE, LIST, VIEW, or UPDATE
  an n8n workflow. Trigger on phrases like: "create a workflow", "build an automation",
  "I want a workflow that does X", "add a new automation", "edit workflow",
  "what workflows do we have", "update this workflow".
  Always use this skill before writing any n8n workflow code — it enforces the correct
  create→validate→save sequence and ensures consistent naming.
---

# n8n Workflow Builder

Handles the full create and update lifecycle for workflows on the shared n8n instance (`vishalmishra.app.n8n.cloud`).

## MCP instance binding (critical)

In Claude Teams (single chat), use the org **n8n Connector** tools only — they must target `vishalmishra.app.n8n.cloud`.

In Claude Code with sub-agents, the orchestrator passes `N8N_MCP_TOOL_PREFIX`; use **only** tools with that prefix. Never use `mcp__n8n-mcp__*` or another n8n MCP family.

After save, confirm with `get_workflow_details(workflowId)` on the same connector before reporting success.

## SDK — live reference only (mandatory)

Before writing **any** workflow code:

1. Call `get_sdk_reference(section="patterns")`.
2. Use **only** the syntax returned (functional `workflow()`, `node()`, `trigger()` — not `WorkflowBuilder` or other patterns from memory).
3. Re-call `get_sdk_reference` if validation fails with SDK/syntax errors.

## Ground Rules

- **Never create without validating first** — always call `validate_workflow` before `create_workflow_from_code`
- **Always describe what you're about to build** — confirm with user before writing code
- **Default project:** `f256nwX37BEaIkA2`

---

## Creating a Workflow

```
Step 1: Confirm intent
- Restate what the workflow will do
- Confirm: trigger, services, expected output

Step 2: Load live SDK (required — first)
- get_sdk_reference(section="patterns") — do not write code until this returns

Step 3: Discover nodes
- search_nodes() for each service involved
- get_node_types() for exact parameter schemas

Step 4: Build workflow code using only Step 2 patterns

Step 5: Validate
- validate_workflow(code) — fix all errors before proceeding

Step 6: Save
- create_workflow_from_code(code, name, description, projectId="f256nwX37BEaIkA2")

Step 7: Verify and confirm
- get_workflow_details(workflowId)
- Report: workflow name, ID, draft status
- Suggest: "Test it now, then say 'publish' when ready"
```

## Naming Convention
`[Category] - [Action] - [Destination]`
Examples: `CRM - Sync Leads - Snowflake`, `Notifications - Slack Alert - On Error`

## After Create/Update
```
✅ Workflow Ready — [Name]
   Status: Draft | ID: [id] | Trigger: [type]
Next: Test it, then publish when ready.
```
