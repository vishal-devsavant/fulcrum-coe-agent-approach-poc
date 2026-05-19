---
name: n8n-workflow-tester
description: >
  Use this skill whenever a team member wants to TEST, RUN, DEBUG, or CHECK the results
  of an n8n workflow. Trigger on phrases like: "test this workflow", "run a test",
  "did the workflow work", "show me the last execution", "why did it fail",
  "run it manually", "check execution history".
---

# n8n Workflow Tester

Safe, structured workflow testing from within Claude chat on `vishalmishra.app.n8n.cloud`.

## MCP instance binding

Use **only** the org n8n Connector (Teams) or the `N8N_MCP_TOOL_PREFIX` passed by the orchestrator (Claude Code). Do not use a different n8n MCP tool family.

If the workflow ID is not found on this instance, report **wrong instance** — do not test elsewhere.

## Two Modes

| Mode | Default? | Real calls? |
|------|----------|-------------|
| Safe Test (pin data) | ✅ Yes | No |
| Live Execute | On request | Yes |

**Default to Safe Test** unless user says "run for real."

## Safe Test Flow
```
1. search_workflows(query) → get workflowId
2. prepare_test_pin_data(workflowId)
3. test_workflow(workflowId, pinData)
4. get_execution(executionId, includeData=true)
5. Report results (see format)
```

## Result Format
```
🧪 Test Results — [Workflow Name]
   Status: ✅ PASSED / ❌ FAILED at [Node]
   
Node Results:
  ✅ [Node] — [plain English summary]

Next: "Say 'publish' when ready" (if passed)
      "Let me fix that for you" (if failed)
```
