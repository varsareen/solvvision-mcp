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
| `sn_risk_risk` | `4f2e0ff383e40310ad3cc4d0deaad3fb` | `__wave1_guard_test__` (RK0020248) | Batch 5 |
| `sn_si_incident` | `2b2e4ff383e40310ad3cc4d0deaad373` | `__wave1_guard_test__` (SIR0010002) | Batch 5 |
| `rm_story` | `cdbdcb7383e40310ad3cc4d0deaad3ce` | `__wave1_guard_test__` (STRY0010001) | Batch 2 |
| `rm_epic` | `e5bd0f7383e40310ad3cc4d0deaad3d1` | `__wave1_guard_test__` (EPIC0010001) | Batch 2 |
| `rm_scrum_task` | `29bd4f7383e40310ad3cc4d0deaad303` | `__wave1_guard_test__` (STSK0011002) | Batch 2 |
| `sn_customerservice_case` | `e4dd4bb383e40310ad3cc4d0deaad373` | `__wave1_guard_test__` (CS0001002) | Batch 2 |
| `sys_hub_flow` | `357e4f3783e40310ad3cc4d0deaad3b9` | `__wave1_guard_test__` flow | Batch 6 |
| `sys_hub_action_type_definition` | `d67e8f3783e40310ad3cc4d0deaad34d` | `__wave1_guard_test__` action | Batch 6 |
| `sp_portal` | `433f87f783e40310ad3cc4d0deaad32f` | `__wave1_guard_test__` portal | Batch 7 |
| `sp_page` | `a95f0ff783e40310ad3cc4d0deaad344` | `__wave1_guard_test__` page | Batch 7 |
| `sp_widget` | `db3f87f783e40310ad3cc4d0deaad3aa` | `__wave1_guard_test__` widget | Batch 7 |
| `sys_rest_message` | `3e9f8b3b83e40310ad3cc4d0deaad346` | `__wave1_guard_test__` REST message | Batch 8 |
| `sys_import_set_run` | `df9f8b3b83e40310ad3cc4d0deaad3e8` | TH0001112 (fake transform run) | Batch 8 |
| `sysevent_register` | `0cafcb3b83e40310ad3cc4d0deaad3c9` | `wave1.guard_test` event registration | Batch 8 |
| `sysevent` | `d0afcb3b83e40310ad3cc4d0deaad3f9` | `incident.created` fired event | Batch 8 |
| `sysevent_email_action` | `cf0093bb83e40310ad3cc4d0deaad396` | `__wave1_guard_test__` notification | Batch 9 |
| `sys_attachment` | `6c10d3bb83e40310ad3cc4d0deaad322` | `wave1_guard_test.txt` attachment | Batch 9 |
| `sys_email` | `781017bb83e40310ad3cc4d0deaad338` | `__wave1_guard_test__` broadcast email | Batch 9 |

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

### Batch 5 — security.ts (19 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_security_incidents` | ✅ Pass | 3 SIR records |
| `get_security_incident` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_security_incident` | ✅ Pass | Created SIR0010002 — cleanup needed |
| `update_security_incident` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_vulnerabilities` | ✅ Pass | 3 CVE entries |
| `get_vulnerability` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `update_vulnerability` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_grc_risks` | ✅ Pass | 3 risk records |
| `get_grc_risk` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_grc_risk` | ✅ Pass | Created RK0020248 — cleanup needed |
| `list_grc_controls` | ✅ Pass | 3 controls |
| `get_threat_intelligence` | ✅ Pass | 3 IOC records |
| `list_security_playbooks` | ⏭️ Skip | sn_si_playbook table absent (partial SecOps plugin) |
| `run_security_playbook` | ⏭️ Skip | sn_si_playbook_execution table absent |
| `get_security_dashboard` | ✅ Pass | INSUFFICIENT_PRIVILEGES — guard removed; PDI role limit |
| `scan_vulnerabilities` | ✅ Pass | INVALID_REQUEST from handler (ci_sys_ids required) — guard removed |
| `list_compliance_policies` | ✅ Pass | 3 policies |
| `get_compliance_assessment` | ✅ Pass | INVALID_REQUEST from handler (sys_id required) — guard removed |
| `list_audit_results` | ⏭️ Skip | sn_audit_result table absent (partial SecOps plugin) |

**Batch 5 total: 16 Pass, 0 Fail, 0 Fishy, 3 Skipped**

### Batch 6 — flow.ts (16 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_flows` | ✅ Pass | 3 flows returned |
| `get_flow` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `trigger_flow` | ❌ Fail | INVALID_REQUEST: table `sys_hub_flow_trigger` does not exist; flows must be triggered via REST API — **escalate to Wave 2** |
| `get_flow_execution` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_flow_executions` | ✅ Pass | 0 records (no executions for fake flow sys_id) |
| `list_subflows` | ❌ Fail | INVALID_REQUEST: table `sys_hub_subflow` does not exist; subflows are stored in `sys_hub_flow` with `type=subflow` — **escalate to Wave 2** |
| `get_subflow` | ❌ Fail | Same wrong table |
| `list_action_instances` | ✅ Pass | 3 action instances |
| `get_process_automation` | ⏭️ Skip | `pa_process` table absent — PAD plugin not installed on PDI |
| `list_process_automations` | ⏭️ Skip | `pa_process` table absent |
| `create_flow` | ✅ Pass | Created `__wave1_guard_test__` flow — cleanup: `357e4f3783e40310ad3cc4d0deaad3b9` |
| `create_subflow` | ❌ Fail | Same wrong table `sys_hub_subflow` |
| `create_flow_action` | ✅ Pass | Created `__wave1_guard_test__` action — cleanup: `d67e8f3783e40310ad3cc4d0deaad34d` |
| `publish_flow` | ✅ Pass | NOT_FOUND (fake sys_id, guard removed) |
| `test_flow` | ❌ Fail | Same wrong table `sys_hub_flow_trigger` |
| `get_flow_error_log` | ✅ Pass | 0 records |

**Batch 6 total: 9 Pass, 5 Fail, 0 Fishy, 2 Skipped**
**New escalations: `trigger_flow`, `test_flow` (wrong trigger mechanism), `list_subflows`, `get_subflow`, `create_subflow` (wrong table sys_hub_subflow)**

### Batch 7 — portal.ts (16 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_portals` | ✅ Pass | 3 portals returned |
| `get_portal` | ✅ Pass | Returned "Service Portal" (url_suffix=sp) |
| `list_portal_pages` | ⚠️ Fishy | 50 records returned for newly-created empty portal; `sp_portal` filter on `sp_page` appears ignored — escalate to Wave 2 |
| `get_portal_page` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_portal_widgets` | ✅ Pass | 3 widgets returned |
| `get_portal_widget` | ✅ Pass | Returned "Cool Clock" widget by id |
| `list_widget_instances` | ✅ Pass | 0 results (fake widget_sys_id) |
| `create_portal` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `433f87f783e40310ad3cc4d0deaad32f` |
| `create_portal_page` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `a95f0ff783e40310ad3cc4d0deaad344` |
| `create_portal_widget` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `db3f87f783e40310ad3cc4d0deaad3aa` |
| `update_portal_widget` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_ux_apps` | ✅ Pass | 3 UX apps returned |
| `get_ux_app` | ✅ Pass | NOT_FOUND (fake name) |
| `list_ux_pages` | ⚠️ Fishy | 22 records returned for fake app_sys_id; `ux_app_config` filter on `sys_ux_page` appears ignored — escalate to Wave 2 |
| `list_portal_themes` | ✅ Pass | 3 themes returned |
| `get_portal_theme` | ✅ Pass | NOT_FOUND (fake sys_id) |

**Batch 7 total: 14 Pass, 0 Fail, 2 Fishy, 0 Skipped**
**Fishy escalations: `list_portal_pages` (sp_portal filter), `list_ux_pages` (ux_app_config filter)**

### Batch 8 — integration.ts (19 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_rest_messages` | ✅ Pass | 3 REST messages returned |
| `get_rest_message` | ✅ Pass | NOT_FOUND (fake name) |
| `list_rest_message_functions` | ✅ Pass | 0 results (fake rest_message_sys_id) |
| `create_rest_message` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `3e9f8b3b83e40310ad3cc4d0deaad346` |
| `list_transform_maps` | ✅ Pass | 3 maps returned |
| `get_transform_map` | ✅ Pass | NOT_FOUND (fake name) |
| `run_transform_map` | ✅ Pass | Created sys_import_set_run TH0001112 — cleanup: `df9f8b3b83e40310ad3cc4d0deaad3e8` |
| `list_transform_field_maps` | ✅ Pass | 0 results (fake transform_map_sys_id) |
| `list_import_sets` | ✅ Pass | 3 import sets returned |
| `get_import_set` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_import_set_row` | ✅ Pass | INVALID_REQUEST (table doesn't exist) — guard removed |
| `list_data_sources` | ✅ Pass | 3 data sources returned |
| `list_event_registry` | ✅ Pass | 3 events returned |
| `get_event_registry_entry` | ⚠️ Fishy | Query uses `name=` but field is `event_name` on sysevent_register; returns first record regardless of filter — escalate to Wave 2 |
| `register_event` | ⚠️ Fishy | SCRIPTING_ENABLED=true; record created but `event_name` is empty — code sets `name` instead of `event_name` — cleanup: `0cafcb3b83e40310ad3cc4d0deaad3c9`; escalate to Wave 2 |
| `fire_event` | ✅ Pass | Created sysevent record — cleanup: `d0afcb3b83e40310ad3cc4d0deaad3f9` |
| `list_event_log` | ✅ Pass | 3 heartbeat events returned |
| `list_oauth_applications` | ✅ Pass | 3 OAuth apps returned |
| `list_credential_aliases` | ✅ Pass | 3 aliases returned |

**Batch 8 total: 17 Pass, 0 Fail, 2 Fishy, 0 Skipped**
**Fishy escalations: `get_event_registry_entry` and `register_event` (wrong field `name` vs `event_name` on sysevent_register)**

### Batch 9 — notification.ts (14 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_notifications` | ✅ Pass | 3 notifications returned |
| `get_notification` | ✅ Pass | NOT_FOUND (fake name) |
| `create_notification` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `cf0093bb83e40310ad3cc4d0deaad396` |
| `update_notification` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_email_logs` | ✅ Pass | 3 email log entries returned |
| `get_email_log` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_attachments` | ✅ Pass | 0 results (no attachments on fake record) |
| `get_attachment_metadata` | ✅ Pass | NOT_FOUND (fake attachment_sys_id) |
| `delete_attachment` | ✅ Pass | NOT_FOUND (fake attachment_sys_id) |
| `upload_attachment` | ✅ Pass | Uploaded 4-byte txt — cleanup: `6c10d3bb83e40310ad3cc4d0deaad322` |
| `list_email_templates` | ✅ Pass | 3 templates returned |
| `list_notification_subscriptions` | ✅ Pass | 0 results |
| `send_emergency_broadcast` | ✅ Pass | Created queued sys_email (fake recipient) — cleanup: `781017bb83e40310ad3cc4d0deaad338` |
| `schedule_notification` | ✅ Pass | NOT_FOUND (fake notification_id) |

**Batch 9 total: 14 Pass, 0 Fail, 0 Fishy, 0 Skipped**
