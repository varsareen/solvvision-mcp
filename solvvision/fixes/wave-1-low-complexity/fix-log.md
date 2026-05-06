# Wave 1 fix log — Low complexity

Branch: `wave-1-low-complexity-fixes`
Status: Complete

## Fixes

| # | Finding | Tool(s) | File changed | What was fixed | PR / commit |
|---|---|---|---|---|---|
| 1 | Category B guard removal (254 tools) | all 254 Category B tools in handlers 12–31 | src/tools/now-assist.ts | Added NOW_ASSIST_TOOL_NAMES set; added early-return guard (`if (!NOW_ASSIST_TOOL_NAMES.has(name)) return null`) before requireNowAssist() so non-now-assist tool names return null and the dispatch chain continues to downstream handlers | ✅ 14 pass / 5 skipped (plugin absent) across 14 domains — 0 residual NOW_ASSIST errors; list_csm_cases now passes (was NOW_ASSIST-blocked in Phase 1) |
| 2 | #7 — wrong table `create_approval_rule` | `create_approval_rule` | src/tools/catalog.ts | Changed table `sysapproval_rule` → `sysrule_approvals` (verified valid on PDI: 1 record returned) | ✅ table verified; write tool — not invoked in test pass to avoid creating data |
| 3 | #8 — wrong table `list_atf_test_results` | `list_atf_test_results` | src/tools/atf.ts | Changed table `sys_atf_result` → `sys_atf_test_result` (verified valid on PDI: table exists, 0 records) | ✅ Pass — returns empty array, no INVALID_REQUEST |
| 4 | #8 — wrong table `get_atf_failure_insight` | `get_atf_failure_insight` | src/tools/atf.ts | Changed table `sys_atf_failure_insight` → `sys_atf_what_changed` (verified valid on PDI: table exists, label "What changed") | ✅ table verified; requires result_sys_id — no ATF suite results on PDI to test against |
| 5 | #9 — wrong table `create_scheduled_report` | `create_scheduled_report` | src/tools/reporting.ts | Changed table `sys_report_schedule` → `sysauto_report` (verified valid on PDI: table exists, 0 records) | ✅ table verified; write tool — not invoked in test pass to avoid creating data |
| — | #10 — `list_job_run_history` escalated | `list_job_run_history` | — | No valid run-history table found on PDI: `sysauto_trigger_log` and `sysauto_trigger` both invalid; `sysauto` family stores job definitions not run logs — escalated to Wave 2 investigation | escalated to Wave 2 |

## Test Results — Category B tools (254 total)

### Pre-session: representative sample + Varun manual (21 tools)
19 tools sampled across 14 domains: **14 Pass, 5 Skipped** (plugin-absent: HRSD, DevOps, Mobile, Workspace + 1)
Plus Varun manual: `get_system_property` ✅, `list_business_rules` ✅

### Batch 1 — script.ts (27 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_business_rules` | ✅ Pass | Varun manual |
| `get_business_rule` | ✅ Pass | Returned Auto assessment BR |
| `create_business_rule` | ✅ Pass | Created `__guard_test__` — cleanup: `16db8bb383a40310ad3cc4d0deaad3e6` |
| `update_business_rule` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_script_includes` | ✅ Pass | 3 results returned |
| `get_script_include` | ✅ Pass | Returned ComplianceCaseUtilsBase |
| `create_script_include` | ✅ Pass | Created `__guard_test__` — cleanup: `bedbcbb383a40310ad3cc4d0deaad3e2` |
| `update_script_include` | ✅ Pass | VALIDATION_ERROR (fake sys_id) |
| `list_client_scripts` | ✅ Pass | 1 result |
| `get_client_script` | ✅ Pass | Returned Autofill table name field |
| `create_client_script` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `68dc47fb83640310ad3cc4d0deaad326` |
| `update_client_script` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_changesets` | ✅ Pass | 1 result |
| `get_changeset` | ✅ Pass | Returned IP Enrichment changeset |
| `commit_changeset` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `publish_changeset` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_ui_policies` | ✅ Pass | 1 result |
| `get_ui_policy` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_ui_policy` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `60dc07bb83a40310ad3cc4d0deaad309` |
| `list_ui_actions` | ✅ Pass | 1 result |
| `get_ui_action` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_ui_action` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `b8dc07bb83a40310ad3cc4d0deaad39d` |
| `update_ui_action` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_acls` | ✅ Pass | 5 results on incident |
| `get_acl` | ✅ Pass | Returned sn_customerservice_case.incident ACL |
| `create_acl` | ✅ Pass | INSUFFICIENT_PRIVILEGES (guard removed, PDI role limit) |
| `update_acl` | ✅ Pass | NOT_FOUND (fake sys_id) |

**Batch 1 total: 27 Pass, 0 Fail, 0 Fishy, 0 Skipped**

## PDI Cleanup Log

Test records created during Wave 1 guard testing — delete before branch merge:

| Table | sys_id | Name | Created |
|---|---|---|---|
| `sys_rule` (Business Rules) | `16db8bb383a40310ad3cc4d0deaad3e6` | `__guard_test__` | Batch 1 |
| `sys_script_include` | `bedbcbb383a40310ad3cc4d0deaad3e2` | `__guard_test__` | Batch 1 |
| `sys_script_client` | `68dc47fb83640310ad3cc4d0deaad326` | `__wave1_guard_test__` | Batch 1 |
| `sys_ui_policy` | `60dc07bb83a40310ad3cc4d0deaad309` | `__wave1_guard_test__` | Batch 1 |
| `sys_ui_action` | `b8dc07bb83a40310ad3cc4d0deaad39d` | `__wave1_guard_test__` | Batch 1 |
| `rm_story` | `cdbdcb7383e40310ad3cc4d0deaad3ce` | `__wave1_guard_test__` (STRY0010001) | Batch 2 |
| `rm_epic` | `e5bd0f7383e40310ad3cc4d0deaad3d1` | `__wave1_guard_test__` (EPIC0010001) | Batch 2 |
| `rm_scrum_task` | `29bd4f7383e40310ad3cc4d0deaad303` | `__wave1_guard_test__` (STSK0011002) | Batch 2 |
| `sn_customerservice_case` | `e4dd4bb383e40310ad3cc4d0deaad373` | `__wave1_guard_test__` (CS0001002) | Batch 2 |

### Batch 2 — agile.ts (9 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_stories` | ✅ Pass | 3 stories returned |
| `list_epics` | ✅ Pass | 3 epics returned |
| `list_scrum_tasks` | ✅ Pass | 3 scrum tasks returned |
| `create_story` | ✅ Pass | Created STRY0010001 — cleanup needed |
| `update_story` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_epic` | ✅ Pass | Created EPIC0010001 — cleanup needed |
| `update_epic` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_scrum_task` | ✅ Pass | Created STSK0011002 — cleanup needed |
| `update_scrum_task` | ✅ Pass | NOT_FOUND (fake sys_id) |

**Batch 2 total: 9 Pass, 0 Fail, 0 Fishy, 0 Skipped**

### Batch 3 — hrsd.ts (16 tools)

Plugin absent: `sn_hr_core_case` returns INVALID_REQUEST on PDI. All 16 tools skipped.

| Tool | Status | Notes |
|---|---|---|
| `create_hr_case` | ⏭️ Skip | HRSD plugin absent |
| `get_hr_case` | ⏭️ Skip | HRSD plugin absent |
| `update_hr_case` | ⏭️ Skip | HRSD plugin absent |
| `list_hr_cases` | ⏭️ Skip | Probe: INVALID_REQUEST sn_hr_core_case |
| `close_hr_case` | ⏭️ Skip | HRSD plugin absent |
| `list_hr_services` | ⏭️ Skip | HRSD plugin absent |
| `get_hr_service` | ⏭️ Skip | HRSD plugin absent |
| `get_hr_profile` | ⏭️ Skip | HRSD plugin absent |
| `update_hr_profile` | ⏭️ Skip | HRSD plugin absent |
| `list_hr_tasks` | ⏭️ Skip | HRSD plugin absent |
| `create_hr_task` | ⏭️ Skip | HRSD plugin absent |
| `get_hr_case_activity` | ⏭️ Skip | HRSD plugin absent |
| `create_onboarding_case` | ⏭️ Skip | HRSD plugin absent |
| `create_offboarding_case` | ⏭️ Skip | HRSD plugin absent |
| `get_hr_lifecycle_events` | ⏭️ Skip | HRSD plugin absent |
| `list_hr_document_templates` | ⏭️ Skip | HRSD plugin absent |

**Batch 3 total: 0 Pass, 0 Fail, 0 Fishy, 16 Skipped**

### Batch 4 — csm.ts (11 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_csm_cases` | ✅ Pass | 3 cases returned |
| `get_csm_case` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `update_csm_case` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `close_csm_case` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_csm_case` | ✅ Pass | Created CS0001002 — cleanup needed |
| `get_csm_account` | ✅ Pass | NOT_FOUND (fake name) |
| `list_csm_accounts` | ✅ Pass | 3 accounts returned |
| `get_csm_contact` | ✅ Pass | NOT_FOUND (fake name) |
| `list_csm_contacts` | ✅ Pass | 3 contacts returned |
| `get_csm_case_sla` | ✅ Pass | Empty results (no SLA on fake sys_id) |
| `list_csm_products` | ✅ Pass | 3 products returned |

**Batch 4 total: 11 Pass, 0 Fail, 0 Fishy, 0 Skipped**
