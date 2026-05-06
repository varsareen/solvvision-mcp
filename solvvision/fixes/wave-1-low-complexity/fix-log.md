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
