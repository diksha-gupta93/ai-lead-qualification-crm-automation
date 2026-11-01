# AI Lead Qualification — Final Acceptance Checklist

Use **synthetic leads only**. Set Postman collection variable `baseUrl` to `http://localhost:5678` for the local Docker instance; `webhookPath` to `webhook` for the published production trigger, or `webhook-test` while listening manually. **The main workflow must be published for production-webhook tests.**

**Before testing:**
- [ ] Confirm test HubSpot environment and test Gmail address; avoid sending to real customers.
- [ ] Inspect and correct `Mark Approved` and `Mark Rejected`: missing `keyName: id` in exported filters.
- [ ] Confirm new test email addresses are not already in `lead_registry`; use unique email suffixes if rerunning.
- [ ] Open n8n Executions and Data Tables; keep HubSpot Contacts and Gmail Sent available.
- [ ] Note that the form uses `company_domain`; Hunter API may fail for unknown domains.

| ID | Test | Expected behavior | Run | Pass | Actual score/ category/ notes |
|---|---|---|---|---|---|
| T01 | T01 - HOT - approve | Approve; check SENT and CONTACTED | [x] | [x] | Score 85/100, HOT category. HubSpot contact created. Approval email received and approved. Personalized email generated. |
| T02 | T02 - WARM - approve | Approve; check SENT and CONTACTED | [x] | [x] | Score 60/100, WARM category. HubSpot contact created. Approval email received and approved. Personalized email generated. |
| T03 | T03 - COLD | Check NURTURE; no approval email | [x] | [x] | Score 35/100, COLD category. HubSpot contact created. Status NURTURE in lead_registry. No email sent. |
| T04 | T04 - REVIEW - unrelated service | Check MANUAL_REVIEW and review notification | [x] | [x] | REVIEW branch executed. Service match LOW. Manual-review email received. |
| T05 | T05 - HOT - reject | Reject; check no outgoing follow-up and APPROVAL_REJECTED | [x] | [x] | Score 100, HOT Category. Approval rejected. Draft status changed to REJECTED. No customer email sent. |
| T06 | T06 - duplicate T01 | Run after T01; check DUPLICATE log and no new draft | [x] | [x] | Duplicate detected in lead_registry. Duplicate logged in lead_processing_log. No repeated email. |
| T07 | T07 - invalid email | Check INVALID log; no Hunter, Groq or HubSpot | [x] | [x] | Validation failed. Reason: Invalid email address. Groq and HubSpot not executed. |
| T08 | T08 - missing message | Check validation errors | [x] | [x] | Validation failed. Reason: Requirement description is required. |
| T09 | T09 - negative budget | Check budget validation error | [x] | [x] | Validation rejected budget of -500. |
| T10 | T10 - no budget | Verify budget=0 points; inspect actual category | [x] | [x] |  Missing budget accepted. Budget score contribution: 0. |
| T11 | T11 - Centralized Error Notification | Check actual Hunter response; may fail before Groq; do not present as fallback success | [x] | [x] | Error Trigger executed. Gmail alert received with failed node and error message. |

## Detailed assertions

- **HOT:** HubSpot score/category; Gmail sales alert; draft created; approval leads to send; registry `CONTACTED` and draft `SENT`.
- **WARM:** HubSpot score/category; sales alert; draft/approval and send; registry `CONTACTED`.
- **COLD:** HubSpot updated; registry `NURTURE`, `COLD`; no draft or customer email.
- **REVIEW:** HubSpot category REVIEW; review notification; registry `MANUAL_REVIEW`; no customer follow-up.
- **DUPLICATE:** run T01 first, then T06 with identical email; log duplicate; no second qualification/draft.
- **INVALID:** log invalid reasons; no downstream enrichment/qualification/CRM actions.
- **HUNTER:** current workflow retries Hunter but has no error branch. API failure is a known limitation, not a successful fallback.
- **Groq:** HTTP Request retries on request failure; malformed JSON in later parser raises error and does not automatically retry the HTTP node.

## Test Run Summary

Date: 2026-10-27

Passed: 11 / 11
Failed: 0 / 11

## Evidence capture

- [x] n8n full workflow canvas screenshot
- [x] HubSpot contact showing category and score
- [x] Gmail approval request
- [x] Error alert email
- [x] Sanitized n8n workflow export

## Notes on score expectations

Expected targets are illustrative. Groq may assign different service fit or authority values based on wording. Check `ai_qualification`, `score_breakdown`, and actual `lead_category` rather than forcing a desired output.

Important: the sample company domain `microsoft.com` is a real public domain but leads/addresses are fictional. Do not represent fictional leads as genuine Microsoft inquiries.
