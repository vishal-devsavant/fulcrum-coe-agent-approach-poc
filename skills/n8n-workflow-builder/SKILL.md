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

Step 2: Discover nodes
- search_nodes() for each service involved
- get_node_types() for exact parameter schemas
- get_sdk_reference(section="patterns")

Step 3: Build workflow code (TypeScript SDK)

Step 4: Validate
- validate_workflow(code) — fix all errors before proceeding

Step 5: Save
- create_workflow_from_code(code, name, description, projectId="f256nwX37BEaIkA2")

Step 6: Confirm
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
