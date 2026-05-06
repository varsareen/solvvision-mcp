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
| `sys_properties` | `d1b0d73f83e40310ad3cc4d0deaad373` | `x.wave1.guard.test.1` | Batch 11 |
| `sys_properties` | _(name-based)_ | `x.wave1.guard.test.2` | Batch 11 |
| `sys_update_set` | `05c09f3f83e40310ad3cc4d0deaad351` | `__wave1_guard_test__` update set | Batch 12 |
| `sys_cs_topic` | `61c09f3f83e40310ad3cc4d0deaad3ca` | `__wave1_guard_test__` VA topic | Batch 13 |
| `sysauto_script` | `ffa257f383280310ad3cc4d0deaad3a2` | `__wave1_guard_test__` scheduled job | Batch 14 |
| `sys_report` | `54b297f383280310ad3cc4d0deaad330` | `__wave1_guard_test__` report | Batch 14 |
| `sysauto_report` | `e3b25bf383280310ad3cc4d0deaad3fa` | scheduled report delivery (fake report_id) | Batch 14 |
| `pa_indicators` | `b3b29bf383280310ad3cc4d0deaad302` | `__wave1_guard_test__` KPI | Batch 14 |
| `alm_asset` | `3dd2dff383280310ad3cc4d0deaad315` | `__wave1_guard_test__` / WAVE1-GUARD-TEST-001 | Batch 15 |
| `change_request` | `0af2973783280310ad3cc4d0deaad395` | CHG0030059 `__wave1_guard_test__ DevOps change` | Batch 16 |
| `sys_app` | `95131f3783280310ad3cc4d0deaad39a` | `__wave1_guard_test__` scope `x_wave1_grd_tst` | Batch 17 |

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

### Batch 10 — performance.ts (15 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_pa_indicators` | ✅ Pass | 3 indicators returned |
| `get_pa_indicator` | ✅ Pass | Returned "% of incidents resolved by first assigned group" |
| `get_pa_scorecard` | ✅ Pass | NOT_FOUND (fake indicator_sys_id) |
| `get_pa_time_series` | ✅ Pass | 0 results (fake indicator_sys_id) |
| `list_pa_breakdowns` | ✅ Pass | 3 breakdowns returned |
| `list_pa_dashboards` | ✅ Pass | 3 dashboards returned |
| `get_pa_dashboard` | ✅ Pass | NOT_FOUND (fake name) |
| `list_homepages` | ❌ Fail | INVALID_REQUEST: table `sys_ui_hp` does not exist — escalate to Wave 2 |
| `list_pa_jobs` | ❌ Fail | INVALID_REQUEST: table `pa_job` does not exist — escalate to Wave 2 |
| `get_pa_job` | ❌ Fail | INVALID_REQUEST: same wrong table `pa_job` |
| `create_dashboard` | ✅ Pass | INSUFFICIENT_PRIVILEGES — guard removed; PDI role limitation |
| `update_dashboard` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `check_table_completeness` | ✅ Pass | Analyzed incident (priority 100%, state 100%, assigned_to 90%) |
| `get_table_record_count` | ✅ Pass | 81 incidents counted |
| `compare_record_counts` | ✅ Pass | Returns counts for incident, change_request, problem |

**Batch 10 total: 12 Pass, 3 Fail, 0 Fishy, 0 Skipped**
**Fail escalations: `list_homepages` (wrong table sys_ui_hp), `list_pa_jobs` and `get_pa_job` (wrong table pa_job)**

### Batch 11 — sys-properties.ts (11 tools, excl. get_system_property — pre-tested)

| Tool | Status | Notes |
|---|---|---|
| `list_system_properties` | ✅ Pass | 3 properties returned |
| `search_system_properties` | ✅ Pass | 3 glide.smtp results |
| `bulk_get_properties` | ✅ Pass | Both props not found on this PDI (correct) |
| `export_properties` | ⚠️ Fishy | category=email filter ignored; returned all 500 props (PDI has no categorized properties — query silently ignored) — escalate to Wave 2 |
| `validate_property` | ✅ Pass | glide.smtp.host not found, returns "would be created as new" |
| `list_property_categories` | ✅ Pass | Only "(uncategorised)" category (998 records) |
| `get_property_history` | ✅ Pass | 3 audit history entries for glide.smtp.host |
| `set_system_property` | ✅ Pass | Created `x.wave1.guard.test.1` — cleanup: `d1b0d73f83e40310ad3cc4d0deaad373` |
| `bulk_set_properties` | ✅ Pass | Created `x.wave1.guard.test.2` — cleanup: name-based delete |
| `import_properties` | ✅ Pass | dry_run=true; showed "create" action (no writes) |
| `delete_system_property` | ✅ Pass | `{deleted: false}` for nonexistent property |

**Batch 11 total: 10 Pass, 0 Fail, 1 Fishy, 0 Skipped**
**Fishy escalation: `export_properties` (category filter silently ignored)**

### Batch 12 — updateset.ts (8 tools)

| Tool | Status | Notes |
|---|---|---|
| `get_current_update_set` | ✅ Pass | 5 in-progress update sets returned |
| `list_update_sets` | ✅ Pass | 3 update sets returned |
| `create_update_set` | ✅ Pass | Created `__wave1_guard_test__` (switch_to=false) — cleanup: `05c09f3f83e40310ad3cc4d0deaad351` |
| `switch_update_set` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `complete_update_set` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `preview_update_set` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `export_update_set` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `ensure_active_update_set` | ✅ Pass | Found existing in-progress update set |

**Batch 12 total: 8 Pass, 0 Fail, 0 Fishy, 0 Skipped**

### Batch 13 — va.ts (7 tools)

| Tool | Status | Notes |
|---|---|---|
| `create_va_topic` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `61c09f3f83e40310ad3cc4d0deaad3ca` |
| `update_va_topic` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `get_va_topic` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_va_topics_full` | ✅ Pass | 3 topics returned |
| `get_va_conversation` | ❌ Fail | INVALID_REQUEST: table `sys_cs_conversation_message` does not exist — escalate to Wave 2 |
| `list_va_conversations` | ✅ Pass | 0 results |
| `list_va_categories` | ❌ Fail | INVALID_REQUEST: table `sys_cs_category` does not exist — escalate to Wave 2 |

**Batch 13 total: 5 Pass, 2 Fail, 0 Fishy, 0 Skipped**
**Fail escalations: `get_va_conversation` (wrong table sys_cs_conversation_message), `list_va_categories` (wrong table sys_cs_category)**

### Batch 14 — reporting.ts (17 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_reports` | ✅ Pass | 3 reports returned |
| `get_report` | ✅ Pass | NOT_FOUND (fake name) |
| `run_aggregate_query` | ✅ Pass | 5 priority groups returned for incident |
| `trend_query` | ✅ Pass | 2-month trend data returned |
| `get_performance_analytics` | ❌ Fail | Fallback hits invalid table `pa_job_log` (INVALID_REQUEST) — escalate to Wave 2 |
| `export_report_data` | ✅ Pass | 3 incidents exported |
| `get_sys_log` | ✅ Pass | QUERY_FAILED (syslog access restricted by role — not a guard error) |
| `list_scheduled_jobs` | ✅ Pass | 3 jobs returned |
| `get_scheduled_job` | ✅ Pass | NOT_FOUND (fake name) |
| `create_scheduled_job` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `ffa257f383280310ad3cc4d0deaad3a2` |
| `update_scheduled_job` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `trigger_scheduled_job` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_report` | ✅ Pass | Created `__wave1_guard_test__` — cleanup: `54b297f383280310ad3cc4d0deaad330` |
| `update_report` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_job_run_history` | ❌ Fail | INVALID_REQUEST: table `sysauto_trigger_log` does not exist — previously escalated to Wave 2 (Fix #10) |
| `create_scheduled_report` | ✅ Pass | Created sysauto_report (Wave 1 fix verified) — cleanup: `e3b25bf383280310ad3cc4d0deaad3fa` |
| `create_kpi` | ✅ Pass | Created pa_indicators `__wave1_guard_test__` — cleanup: `b3b29bf383280310ad3cc4d0deaad302` |

**Batch 14 total: 15 Pass, 2 Fail, 0 Fishy, 0 Skipped**
**Fail escalations: `get_performance_analytics` (fallback hits invalid table pa_job_log), `list_job_run_history` (invalid table sysauto_trigger_log — known, Fix #10)**

### Batch 15 — itam.ts (10 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_assets` | ✅ Pass | 3 assets returned |
| `get_asset` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `create_asset` | ✅ Pass | Created `WAVE1-GUARD-TEST-001` — cleanup: `3dd2dff383280310ad3cc4d0deaad315` |
| `update_asset` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `retire_asset` | ✅ Pass | NOT_FOUND (fake sys_id) |
| `list_software_licenses` | ✅ Pass | 3 licenses returned |
| `get_license_compliance` | ✅ Pass | 36 licenses analyzed |
| `list_asset_contracts` | ✅ Pass | 3 contracts returned |
| `track_asset_lifecycle` | ✅ Pass | NOT_FOUND (fake asset_tag) |
| `get_license_optimization` | ✅ Pass | 36 license recommendations returned |

**Batch 15 total: 10 Pass, 0 Fail, 0 Fishy, 0 Skipped**

### Batch 16 — devops.ts (7 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_devops_pipelines` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_pipeline` — DevOps plugin absent |
| `get_devops_pipeline` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_pipeline` — DevOps plugin absent |
| `list_deployments` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_deploy_task` — DevOps plugin absent |
| `get_deployment` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_deploy_task` — DevOps plugin absent |
| `create_devops_change` | ✅ Pass | Creates standard `change_request` (no DevOps plugin needed) — cleanup: `0af2973783280310ad3cc4d0deaad395` CHG0030059 |
| `track_deployment` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_deploy_task` — DevOps plugin absent |
| `get_devops_insights` | ⏭️ Skip | INVALID_REQUEST: table `sn_devops_deploy_task` — DevOps plugin absent |

**Batch 16 total: 1 Pass, 0 Fail, 0 Fishy, 6 Skipped (DevOps plugin absent)**

### Batch 17 — app-studio.ts (4 tools)

| Tool | Status | Notes |
|---|---|---|
| `list_scoped_apps` | ✅ Pass | 3 apps returned (guard removed — was Phase 1 NOW_ASSIST-blocked) |
| `get_scoped_app` | ✅ Pass | NOT_FOUND (fake scope) |
| `create_scoped_app` | ✅ Pass | Created `__wave1_guard_test__` scope `x_wave1_grd_tst` — cleanup: `95131f3783280310ad3cc4d0deaad39a` |
| `update_scoped_app` | ✅ Pass | NOT_FOUND (fake sys_id) |

**Batch 17 total: 4 Pass, 0 Fail, 0 Fishy, 0 Skipped**

### Batch 18 — ml.ts (10 tools)

| Tool | Status | Notes |
|---|---|---|
| `ml_predict_change_risk` | ✅ Pass | Historical analysis returned (predicted_risk: low) |
| `ml_detect_anomalies` | ✅ Pass | 0 anomalies in 3 incidents over 7 days |
| `ml_forecast_incidents` | ✅ Pass | Forecast returned (avg_daily_rate: 0.1) |
| `ml_train_incident_classifier` | ✅ Pass | PI solution found; training endpoint unavailable — returns training_failed gracefully |
| `ml_train_change_risk` | ✅ Pass | Same graceful failure path |
| `ml_train_anomaly_detector` | ✅ Pass | Returns queue message without API call |
| `ml_evaluate_model` | ✅ Pass | NOT_FOUND (fake model_sys_id on ml_solution) |
| `ml_model_training_history` | ❌ Fail | INVALID_REQUEST: table `ml_solution_version` does not exist — escalate to Wave 2 |
| `ml_virtual_agent_nlu` | ✅ Pass | 0 conversations in 7-day window |
| `ml_process_optimization` | ✅ Pass | 1 resolved incident analyzed, bottleneck LOW |

**Batch 18 total: 9 Pass, 1 Fail, 0 Fishy, 0 Skipped**
**Fail escalation: `ml_model_training_history` (invalid table ml_solution_version)**
