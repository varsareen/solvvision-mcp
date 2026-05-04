# Stubs & caveats

Tools in the upstream fork that don't fully do what their name implies.

Four severity tiers, most concerning first.

| Tier | Meaning |
|---|---|
| 1 | **Pure stub** — returns fake data, makes no HTTP call |
| 2 | **Not implemented** — handler throws `NOT_IMPLEMENTED` |
| 3 | **Misleading name** — works, but narrower scope than the name suggests |
| 4 | **Hard-coded table alias** — works, but is a thin wrapper over `query_records` |

## Tier 1 — pure stubs (2 tools)

These return placeholder JSON without touching ServiceNow.

### `create_solution_package` (deployment)

**Symptom:** Returns placeholder JSON saying "Solution packaging requires
Store app or manual export". Makes no HTTP call.

**Recommendation:** Skip in tests. Document as a gap. If solution
packaging is needed, do it manually via Studio / Store app, or extend the
fork to call the real API.

### `ml_train_anomaly_detector` (ml)

**Symptom:** Returns placeholder JSON saying "Configure anomaly detection
models via ML Workbench". Makes no HTTP call.

**Recommendation:** Skip in tests. Document as a gap. ML Workbench
operations require the real Predictive Intelligence APIs; this tool
does not call them.

## Tier 2 — not implemented (1 tool)

### `natural_language_update` (core)

**Symptom:** Tool exists and registers, but the handler throws
`ServiceNowError("NOT_IMPLEMENTED")`. Calling it always errors.

**Recommendation:** Skip in tests. Document as not-yet-built. If
natural-language updates matter, this is a candidate for the
fork-extension thread.

## Tier 3 — misleading names (2 tools)

These work, but do less than their names imply.

### `natural_language_search` (core)

**Symptom:** Name implies free-form NL search over any table.
Implementation is hard-coded to the `incident` table only — calls
`queryRecords` with `table=incident` regardless of what you ask it about.

**Recommendation:** Use with eyes open: only useful for incident search.
For other tables use `query_records` directly. Consider renaming to
`natural_language_search_incidents` in the fork.

### `nlq_query` (now-assist)

**Symptom:** Generic dispatcher to a Now Assist NLQ endpoint. Whether it
works depends entirely on whether the Now Assist / NLQ plugin is enabled
in the target instance. Free PDIs may not have it.

**Recommendation:** Probe once against the target instance to check
plugin status before relying on it. Out of scope for Phase 1 validation.

## Tier 4 — hard-coded table aliases (7 tools)

These work, but are one-line aliases over `query_records` on a specific
table. Not bugs — they exist for LLM discoverability — but they offer no
capability beyond `query_records` and are worth knowing about.

| Tool | Aliased table |
|---|---|
| `get_user` | `sys_user` |
| `get_group` | `sys_user_group` |
| `search_cmdb_ci` | `cmdb_ci` |
| `list_relationships` | `cmdb_rel_ci` |
| `list_discovery_schedules` | `discovery_schedule` |
| `list_mid_servers` | `ecc_agent` |
| `list_active_events` | `em_event` |

**Recommendation:** Keep — they help the LLM find capability by name.
Just be aware they don't add anything beyond `query_records`.

## Findings from Phase 1 validation (2026-05-04, dev229998)

### NOW_ASSIST_ENABLED guard — 20 of 30 tools blocked (config caveat)

**Symptom:** 20 of 30 Phase 1 checklist tools return:
```
Error: Now Assist / AI features are disabled.
Set NOW_ASSIST_ENABLED=true to enable. (Code: NOW_ASSIST_NOT_ENABLED)
```

**Scope:** The guard is applied broadly across many handler categories — not
only AI/NLQ tools. Affected tools include basic reads such as
`get_system_property`, `get_table_record_count`, `list_update_sets`,
`list_portals`, `list_notifications`, `list_assets`, `find_artifact`, and
write tools including `create_story`. The 10 tools that worked without the
guard are: `create_incident`, `query_records`, `list_users`, `create_problem`,
`create_change_request`, `list_my_tasks`, `list_catalog_items`,
`list_atf_suites`, `list_reports`, and `close_incident`/`resolve_problem`.

**Solvvision decision:** `NOW_ASSIST_ENABLED=false` is intentional and
permanent in the Solvvision config. Tools blocked by this guard are
classified ⏭️ Skipped (config pre-condition not met), not ❌ Fail.

**Impact on Phase 2:** Any scripted harness run against Solvvision instances
must either:
(a) Set `NOW_ASSIST_ENABLED=true` to test the full tool surface, or
(b) Accept that 20+ tools are untestable under the current config and
    scope Phase 2 to only the tools confirmed working in Phase 1.

**Validation run:** #2 — https://github.com/varsareen/solvvision-mcp/issues/2

---

### `execute_background_script` endpoint — untested (config caveat)

**Symptom:** The `NOW_ASSIST_ENABLED` guard fires before the handler reaches
its HTTP call, so the predicted failure on `/api/now/sp/background_script`
(not a public ServiceNow REST endpoint) was not confirmed or denied.

**Recommendation:** Test with `NOW_ASSIST_ENABLED=true` in a future run to
confirm whether the endpoint exists. Pre-run source review suggests it will
404 — treat as a suspected stub until confirmed.

---

### `search_knowledge` keyword relevance — fishy (tool caveat)

**Symptom:** Query for "email" returned 3 articles with no apparent
relation to email: "Baselining Architecture Overview", "Breach Insight
Templates", "PerfMetricsBaselineCalculator". The keyword may be applied only
to article body text (not returned in the response), or may be dropped
entirely, returning the first N articles by insertion order.

**Finding issue:** #3

**Recommendation:** Verify whether `search_knowledge` uses `sysparm_search`
(full-text) or a `LIKE` filter on `short_description`. If keyword is applied
only to body, responses will always look irrelevant from short_description
alone — add `text` or `body` to returned fields for transparency.

---

### `create_problem` schema — missing impact/urgency (schema caveat)

**Symptom:** The `create_problem` schema exposes `priority` but not
`impact` or `urgency`. Since ServiceNow computes problem priority from
impact × urgency, passing `priority: 2` results in `priority: 5` on the
created record (impact and urgency defaulted to 3 each).

**Recommendation:** Add `impact` and `urgency` parameters to `create_problem`
schema (matching `create_incident`) so priority is set as intended.

---

### `close_change_request` — INSUFFICIENT_PRIVILEGES on Draft changes

**Symptom:** Calling `close_change_request` on a normal change in Draft
state (state=`-5`) returns `INSUFFICIENT_PRIVILEGES`. The change workflow
requires progression through Submit → Approve → Implement before closure
is permitted.

**Recommendation:** For cleanup of Draft test changes, use the PDI UI to
cancel rather than the MCP tool. Alternatively, `update_change_request` to
state=`cancelled` may work — to be tested in a future run.
