---
name: Validation run
about: Run the Solvvision MCP validation checklist against a ServiceNow PDI
title: 'Validation run — <PDI> — <YYYY-MM-DD>'
labels: ['validation', 'phase-1']
---

# Validation run

| Field | Value |
|---|---|
| Target PDI | |
| Date | |
| Phase | 1 (manual prompts in Claude Code) |
| Tester | |
| Fork commit | |

## Pre-flight

- [ ] `.env` points to correct PDI
- [ ] Basic auth credentials work (curl test returns 200)
- [ ] All `WRITE_*_ENABLED` flags = `true`
- [ ] `SCRIPTING_ENABLED=true` (needed for S4.2)
- [ ] Claude Code restarted after `.env` change
- [ ] Solvvision MCP tools visible when typing `/` in Claude Code

---

## Story 1 — Incident triage day

### S1.1 · `create_incident` (write)

**Prompt:** Create a new incident. Short description: `[MCP-VALIDATION-<DATE>] Email service intermittently slow`. Category: software. Impact: 2. Urgency: 2. Caller: admin.

**Watch for:** WRITE flag off (403); urgency schema mismatch (declared number, expects string); silent success without number returned.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S1.2 · `query_records` (read)

**Prompt:** Query the incident table. Show me the 5 most recently updated active incidents. Fields: `number`, `short_description`, `priority`, `assigned_to`.

**Watch for:** Empty list on a seeded PDI is suspicious; 1000+ rows means limit not honoured; missing fields in response.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S1.3 · `list_users` (read)

**Prompt:** Find users whose name contains `admin`. Show their email, title, and active status.

**Watch for:** Empty results (filter dropped); all users returned (filter ignored).

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S1.4 · `list_csm_cases` (read)

**Prompt:** List Customer Service cases that are currently open. Top 5 by priority.

**Watch for:** Real cases OR clean empty list — both valid. Error like `table sn_customerservice_case not found` = CSM plugin not activated, which is a finding not a bug.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S1.5 · `search_knowledge` (read)

**Prompt:** Search the knowledge base for articles about `email`. Top 3 hits.

**Watch for:** Up to 3 KB articles with `KB...` numbers, `short_description`, `workflow_state`. Empty result suggests wrong table (`kb_knowledge` vs `kb_knowledge_base`) or no seed data.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S1.6 · `create_problem` (write)

**Prompt:** Create a problem record. Short description: `[MCP-VALIDATION-<DATE>] Investigating recurring email slowness`. Impact: 2. Urgency: 2.

**Watch for:** `WRITE_PROBLEM_ENABLED` may be a separate flag — check if write fails with 403.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Story 2 — Change management cycle

### S2.1 · `create_change_request` (write)

**Prompt:** Create a normal change request. Short description: `[MCP-VALIDATION-<DATE>] Restart email queue on EMAIL-PROD-01`. Type: normal. Risk: low. Impact: 3.

**Watch for:** PDIs sometimes auto-cancel changes via CAB workflow — note state on creation, but it's not strictly a tool failure.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S2.2 · `list_my_tasks` (read)

**Prompt:** List the tasks currently assigned to me.

**Watch for:** Empty is fine — admin rarely has assigned tasks on a fresh PDI. If ALL tasks are returned (not filtered), filter is being dropped.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S2.3 · `list_workspaces` (read)

**Prompt:** List the configurable agent workspaces in this instance.

**Watch for:** Several seeded workspaces (CSM, ITSM) expected. Empty list — cross-check `sys_aw_master_config` directly.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S2.4 · `list_flows` (read)

**Prompt:** List Flow Designer flows in this instance. Top 10.

**Watch for:** ~10+ flows expected from `sys_hub_flow`. Empty result is suspicious on a stock PDI.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Story 3 — Catalog request flow

### S3.1 · `list_catalog_items` (read)

**Prompt:** List the top 10 service catalog items available in this instance.

**Watch for:** ~10 catalog items with name, category, sys_id. Empty = seed data missing or wrong table.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S3.2 · `list_portals` (read)

**Prompt:** List the Service Portals configured in this instance.

**Watch for:** At least the default `sp` portal expected. Empty list highly suspicious — `sp` is on every instance.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S3.3 · `list_notifications` (read)

**Prompt:** List 5 active email notifications. Show name, table, and trigger condition.

**Watch for:** Definitions from `sysevent_email_action` — empty is suspicious, this table has seed data.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S3.4 · `list_va_topics_full` (read)

**Prompt:** List the Virtual Agent conversation topics in this instance.

**Watch for:** Topics OR plugin-not-activated error. Either is a valid finding — note exactly which plugins are missing.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Story 4 — Platform admin & scripting

### S4.1 · `get_system_property` (read)

**Prompt:** Read the system property `glide.product.description` and tell me its value.

**Watch for:** String value (e.g. "ServiceNow Platform"). Empty value is fine if genuinely unset — confirm metadata fields populated.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S4.2 · `execute_background_script` (script) — **THE FINDING TEST**

> ⚠️ Based on source review, this handler calls `/api/now/sp/background_script` which doesn't appear to be a real ServiceNow REST endpoint. Predicted to fail. If it works, that's also a meaningful finding worth documenting.

**Prompt:** Execute this background script: `gs.info('MCP validation ping');`

**Watch for:** Tool's claim is "script ran." Predicted reality is `{action: "failed", error: "..."}` with 404 / endpoint not found. HTTP 401 = endpoint exists but wrong auth. HTML response = endpoint is a UI page, not REST API. If it actually works, document the response carefully.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S4.3 · `list_update_sets` (read)

**Prompt:** List update sets currently in progress.

**Watch for:** At least the "Default" update set per scope. Empty is suspicious — Default always exists.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S4.4 · `list_security_incidents` (read)

**Prompt:** List security incidents in this instance.

**Watch for:** Rows from `sn_si_incident` OR clean empty / table-not-found if SecOps plugin off. Plugin status is the finding here.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Story 5 — Asset & DevOps adjacent

### S5.1 · `list_assets` (read)

**Prompt:** List 5 IT hardware assets in this instance.

**Watch for:** 5 rows from `alm_hardware`. Empty is suspicious — laptop/monitor demo data ships seeded.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S5.2 · `find_artifact` (read)

**Prompt:** Find business rules whose name contains `incident`.

**Watch for:** List of business rule names with sys_ids. Empty result is highly suspicious — every PDI has incident BRs.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S5.3 · `list_devops_pipelines` (read)

**Prompt:** List DevOps pipelines registered in this instance.

**Watch for:** Likely empty — DevOps Change Velocity plugin rarely on free PDIs. Table-not-found is the expected finding.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S5.4 · `list_hr_cases` (read)

**Prompt:** List HR cases in this instance, top 5.

**Watch for:** Likely empty / table-not-found — HRSD plugin rarely on free PDIs.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S5.5 · `list_mobile_app_configs` (read)

**Prompt:** List mobile app configurations in this instance.

**Watch for:** Seeded mobile apps (Now Mobile, Agent, Onboarding) present everywhere. Empty is genuinely suspicious.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Story 6 — Quality, reporting & engineering

### S6.1 · `list_atf_suites` (read)

**Prompt:** List the ATF test suites in this instance.

**Watch for:** 5–20 seeded suites with name, sys_id, test count. Empty is suspicious on stock PDI.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.2 · `get_table_record_count` (read)

**Prompt:** How many records does the incident table have?

**Watch for:** Number, typically low thousands (50–500 + your test records). Returns 0 — cross-check with S1.2.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.3 · `list_reports` (read)

**Prompt:** List saved reports in this instance, top 10.

**Watch for:** 10 rows from `sys_report`. Empty is suspicious — `sys_report` ships with dozens seeded.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.4 · `ml_forecast_incidents` (read)

**Prompt:** Forecast incident volume for the next 7 days.

**Watch for:** Tool's claim is time-series forecast. Predicted reality: error (Predictive Intelligence plugin off / model not trained). Plugin-status finding.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.5 · `create_story` (write)

**Prompt:** Create an agile story. Short description: `[MCP-VALIDATION-<DATE>] Validate MCP tool coverage for agile module`. Story points: 3. State: ready.

**Watch for:** Story number and sys_id from `rm_story`. Plugin-not-activated → SAFe / SDLC scrum plugin off on this PDI.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.6 · `list_scoped_apps` (read)

**Prompt:** List scoped (custom) applications installed in this instance.

**Watch for:** List of `sys_app` records — ServiceNow's own scoped apps + demos. Empty is highly suspicious.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

### S6.7 · `list_rest_messages` (read)

**Prompt:** List the outbound REST messages configured in this instance.

**Watch for:** Seeded REST integrations (AWS, Azure samples often present). Empty is suspicious — `sys_rest_message` has seed data.

- [ ] ✅ Pass
- [ ] ❌ Fail
- [ ] ⚠️ Fishy
- [ ] ⏭️ Skipped

**Notes:**

---

## Headline findings

Fill in after the run.

- **S4.2 `execute_background_script`:** Did it work or fail? Exact error message:
- **PDI plugins missing on this instance:**
- **Surprising or hardcoded-looking data:**
- **Schema validation errors (e.g. urgency type):**
- **LLM tool selection issues** — did Claude Code pick a different tool than the one targeted?

## Result tally

- ✅ Pass: __ / 30
- ❌ Fail: __ / 30
- ⚠️ Fishy: __ / 30
- ⏭️ Skipped: __ / 30

## Cleanup confirmed

- [ ] Test incident deleted (S1.1)
- [ ] Test problem deleted (S1.6)
- [ ] Test change deleted (S2.1)
- [ ] Test agile story deleted (S6.5)
- [ ] Any S4.2 artefacts (likely none)

## Next steps

- [ ] Update `solvvision/runs/README.md` with link to this issue
- [ ] Update `solvvision/stubs-and-caveats.md` if new findings emerged
- [ ] Decide whether to proceed to Phase 2
