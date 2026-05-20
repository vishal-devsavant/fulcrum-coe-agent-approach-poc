# Testing Report

**Workflow:** Demo - User Details Form - Google Doc (Mock)
**Workflow ID:** 5IApa6fRHNRgwAOu
**Execution ID:** 8
**Tested:** 2026-05-20T09:35:41Z
**Mode:** Manual execution with form input
**Result:** PASSED

## Mock Data Used
| Field | Value |
|-------|-------|
| Full Name | Jane Smith |
| Email | jane.smith@fulcrum.com |
| Department | Engineering |
| Role | Developer |

## Node Results
| Node | Status | Output Summary |
|------|--------|----------------|
| User Details Form | Pass | All four fields returned with exact mock values |
| Save to Google Doc (Mock) | Pass | mock: true, status: success, documentTitle: Fulcrum User Register, appendedContent contained all four field values pipe-delimited, message referenced Jane Smith |

## Verification Results
| Check | Result |
|-------|--------|
| Execution completed without errors | PASS |
| Timestamp is today (2026-05-20) | PASS — startedAt: 2026-05-20T09:35:41.562Z |
| All four form fields flow through correctly | PASS |
| Mock node status = success | PASS |
| appendedContent contains all four field values | PASS |
| message references submitter name | PASS |

## Issues Found
None — two earlier test runs failed due to a Code node template-literal serialization bug in n8n's task runner. Fixed by switching to the mutate-and-return-item pattern (`$input.item.json.x = y; return $input.item`). Third run (execution ID 8) passed cleanly.

## Production Readiness Notes
Replace the Code node with a real Google Docs node and configure Google OAuth2 credentials before going live.
