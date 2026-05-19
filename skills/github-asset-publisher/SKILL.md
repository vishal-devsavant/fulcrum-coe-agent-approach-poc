---
name: github-asset-publisher
description: >
  Use this skill whenever a team member wants to PUBLISH, SUBMIT, or DEPLOY an n8n workflow
  for review. Trigger on: "publish this workflow", "raise a PR", "submit for review",
  "I'm done, submit it", "push to production", "what's the status of my workflow PR".
  Always confirm testing has been completed before raising a PR.
---

# GitHub Asset Publisher

Handles workflow submission: n8n draft → GitHub PR → Admin review → Auto-publish on merge.

## MCP instance binding

- **n8n reads:** org n8n Connector or orchestrator-provided `N8N_MCP_TOOL_PREFIX` only — instance `vishalmishra.app.n8n.cloud`. Never use another n8n MCP prefix.
- **GitHub:** org GitHub Connector only — repo `devsavant/fulcrum-coe`.

## Pre-Flight (Always Check First)
```
1. Workflow exists: search_workflows(query)
2. Has been tested: search_executions(workflowId, limit=1)
   - No executions → "Test it first"
   - Last failed → "Fix and re-test first"
```

## PR Creation
```
1. get_workflow_details(workflowId)
2. create_branch: workflow/<kebab-name>-<YYYYMMDD>
3. Commit JSON to: workflows/<name>.json
4. create_pull_request with standard template
5. Report PR link to user
```

## Response
```
✅ PR Raised — [Workflow Name]
   Branch: workflow/[name]-[date]
   PR:     [URL]
   
What's next:
  Admin reviews on GitHub → approves → workflow goes live in n8n automatically.
```
