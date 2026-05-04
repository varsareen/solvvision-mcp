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

## Findings expected to be added during validation

The Phase 1 validation may surface additional caveats. Expected categories:

- **Tools calling fictional endpoints** — e.g. `execute_background_script`
  appears to call `/api/now/sp/background_script` which is not a public
  ServiceNow REST API. To be confirmed during S4.2 of the Phase 1 run.
- **Tools requiring undocumented plugin entitlements** — likely surfaces
  during Story 5 (DevOps, HRSD) and S6.4 (ML).
- **Schema mismatches** — e.g. the `urgency` field in `create_incident` is
  declared as `number` in the input schema but ServiceNow's REST API expects
  a string. To be confirmed during S1.1.
