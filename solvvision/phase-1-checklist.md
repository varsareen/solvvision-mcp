# Phase 1 validation checklist — reference copy

> This is a static reference copy of the Phase 1 prompt list. Actual runs
> use the [Validation run issue template](../.github/ISSUE_TEMPLATE/validation-run.md)
> which seeds new GitHub Issues with this same content plus tickbox UI.
>
> Maintain this file in sync with the issue template when prompts change.

## Coverage

- 30 tools, one per working domain (now-assist excluded)
- 6 themed mini-stories
- 25 reads, 5 writes/script
- All test records tagged `[MCP-VALIDATION-<DATE>]` for cleanup

## Stories

### Story 1 — Incident triage day (6 tools)

| Step | Tool | Mode |
|---|---|---|
| S1.1 | `create_incident` | write |
| S1.2 | `query_records` | read |
| S1.3 | `list_users` | read |
| S1.4 | `list_csm_cases` | read |
| S1.5 | `search_knowledge` | read |
| S1.6 | `create_problem` | write |

### Story 2 — Change management cycle (4 tools)

| Step | Tool | Mode |
|---|---|---|
| S2.1 | `create_change_request` | write |
| S2.2 | `list_my_tasks` | read |
| S2.3 | `list_workspaces` | read |
| S2.4 | `list_flows` | read |

### Story 3 — Catalog request flow (4 tools)

| Step | Tool | Mode |
|---|---|---|
| S3.1 | `list_catalog_items` | read |
| S3.2 | `list_portals` | read |
| S3.3 | `list_notifications` | read |
| S3.4 | `list_va_topics_full` | read |

### Story 4 — Platform admin & scripting (4 tools)

| Step | Tool | Mode |
|---|---|---|
| S4.1 | `get_system_property` | read |
| S4.2 | `execute_background_script` ⚠️ | script |
| S4.3 | `list_update_sets` | read |
| S4.4 | `list_security_incidents` | read |

> ⚠️ **S4.2 is the headline finding test.** Source review shows the handler
> calls `/api/now/sp/background_script`, which is not a public ServiceNow
> REST endpoint. Predicted to fail; if it works, also a finding.

### Story 5 — Asset & DevOps adjacent (5 tools)

| Step | Tool | Mode |
|---|---|---|
| S5.1 | `list_assets` | read |
| S5.2 | `find_artifact` | read |
| S5.3 | `list_devops_pipelines` | read |
| S5.4 | `list_hr_cases` | read |
| S5.5 | `list_mobile_app_configs` | read |

### Story 6 — Quality, reporting & engineering (7 tools)

| Step | Tool | Mode |
|---|---|---|
| S6.1 | `list_atf_suites` | read |
| S6.2 | `get_table_record_count` | read |
| S6.3 | `list_reports` | read |
| S6.4 | `ml_forecast_incidents` | read |
| S6.5 | `create_story` | write |
| S6.6 | `list_scoped_apps` | read |
| S6.7 | `list_rest_messages` | read |

## Pre-flight

Before each run:

- `.env` points to correct PDI
- Basic auth credentials work
- All `WRITE_*_ENABLED` flags = true
- `SCRIPTING_ENABLED=true` (for S4.2)
- Claude Code restarted after `.env` change
- Solvvision MCP tools visible in `/` menu

## Cleanup

After each run, delete records where `description` or `short_description`
contains `[MCP-VALIDATION-<DATE>]`:

- 1 incident (from S1.1)
- 1 problem (from S1.6)
- 1 change request (from S2.1)
- 1 agile story (from S6.5)

PDI hibernation will eventually wipe these, but explicit cleanup is tidier.
