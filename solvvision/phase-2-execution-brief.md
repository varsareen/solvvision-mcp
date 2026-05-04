# Brief for Claude Code — Phase 2 validation execution

## Context

Phase 1 (30-prompt manual checklist) is complete. This is Phase 2 — a
fully automated harness that covers the remaining in-scope tools not
tested in Phase 1.

Repo: `varsareen/solvvision-mcp`
Working dir: `~/Library/CloudStorage/OneDrive-SOLVVision/Solvvision Projects/claude-code-rollout/solvvision-mcp`
PDI: `dev229998.service-now.com`
Mode: **fully unattended, --dangerously-skip-permissions**

Read `solvvision/stubs-and-caveats.md` before starting — it contains
known issues that affect classification.

---

## Scope

### Excluded tools (do not test)

| Category | Tools | Reason |
|---|---|---|
| Now Assist API | All 15 tools in `now-assist.ts` + any handler with `Now Assist API` in API column | `NOW_ASSIST_ENABLED=false` by design — Solvvision config, permanent |
| Stubs | `create_solution_package`, `ml_train_anomaly_detector` | Known no-ops, return placeholder data without calling ServiceNow |
| NOT_IMPLEMENTED | `natural_language_update` | Known throw, documented caveat |
| Already passing in Phase 1 | `create_incident`, `query_records`, `list_users`, `create_problem`, `create_change_request`, `list_my_tasks`, `list_catalog_items`, `list_atf_suites`, `list_reports` | 9 confirmed passes, no need to retest |
| Phase 1 Skipped (NOW_ASSIST guard) | `list_csm_cases` + any tool that hit NOW_ASSIST_ENABLED guard in Phase 1 | Same config, same result expected |

### In-scope tools

All remaining tools not in the exclusion list above, split into two passes:

**Pass A — Read tools (no data written to PDI)**
All GET-only tools. Run first. Target: ~180 tools.

**Pass B — Write tools (data written to PDI)**
All POST/PATCH/DELETE tools with WRITE_ENABLED or SCRIPTING_ENABLED guards.
Run after Pass A completes. Every record created must be tagged
`[MCP-VALIDATION-PHASE2-<DATE>]` in its short_description or name field.
Target: ~80 tools.

Run Pass A and Pass B in the same session, back to back, with no pause
between them unless an unexpected error requires investigation.

---

## Setup — do these before the first tool invocation

### Step 1 — Open the Phase 2 validation run issue

```bash
COMMIT_SHA=$(git rev-parse --short HEAD)
DATE=$(date +%Y-%m-%d)

gh issue create \
  --title "Phase 2 validation run — dev229998 — ${DATE}" \
  --body "## Phase 2 automated harness run

| Field | Value |
|---|---|
| PDI | dev229998.service-now.com |
| Date | ${DATE} |
| Fork commit | ${COMMIT_SHA} |
| Scope | All in-scope tools excluding Now Assist, stubs, NOT_IMPLEMENTED, Phase 1 passes |
| Mode | Fully automated |

## Pass A — Read tools

_Results appended during run_

## Pass B — Write tools

_Results appended during run_

## Tally

_Filled at end_

## Headline findings

_Filled at end_
" \
  --label validation \
  --label phase-2
```

Save the issue number — you'll need it throughout.

### Step 2 — Initialise the run log

```bash
DATE=$(date +%Y-%m-%d)
cat > solvvision/runs/phase-2-run-${DATE}.md << 'EOF'
# Phase 2 run log

| Tool | Domain | Pass | Classification | Notes |
|------|--------|------|----------------|-------|
EOF

git add solvvision/runs/phase-2-run-${DATE}.md
git commit -m "Start Phase 2 run log — ${DATE}"
git push origin main
```

### Step 3 — Confirm env

Verify MCP server is live and `.env` is correct:
- Call `get_current_instance` — should return `dev229998.service-now.com`
- Confirm `WRITE_ENABLED=true` and `SCRIPTING_ENABLED=true` in `.env`

Print:
```
=== PHASE 2 SETUP COMPLETE ===
Issue: #<N>
Run log: solvvision/runs/phase-2-run-<DATE>.md
MCP instance: dev229998.service-now.com
Starting Pass A (read tools)...
```

Then immediately begin Pass A with no further confirmation needed.

---

## Run loop

### For each tool in scope:

#### a) Invoke the tool

Use a minimal valid payload:
- Tools with no required args: call with empty args `{}`
- Tools requiring a sys_id: first call the corresponding list tool to get
  a real sys_id from the PDI, then call the get tool with that sys_id.
  If the list returns empty, classify as Fishy and note it.
- Tools requiring a name/number: use a known value from Phase 1 where
  possible (e.g. `glide.product.description` for sys-properties tools),
  otherwise use a sensible default

#### b) Classify

- ✅ **Pass** — HTTP 200, expected response shape, real data
- ❌ **Fail** — HTTP error, exception, transport failure, or
  NOW_ASSIST_ENABLED guard on a non-Now-Assist tool
- ⚠️ **Fishy** — 200 but empty when data should exist, clearly wrong
  shape, hardcoded-looking values, or keyword ignored (like Phase 1
  `search_knowledge`)
- ⏭️ **Skipped** — plugin not enabled on PDI (HRSD, CSM, DevOps,
  Predictive Intelligence), or write flag gate on a read tool

#### c) Record immediately — no batching

After every tool, before moving to the next:

**1. Append to run log:**
```bash
echo "| \`<tool>\` | <domain> | <pass_number> | <emoji> <classification> | <one-line note> |" \
  >> solvvision/runs/phase-2-run-${DATE}.md
```

**2. Commit every 10 tools** (not every tool — reduces noise):
```bash
# Every 10th tool:
git add solvvision/runs/phase-2-run-${DATE}.md
git commit -m "Phase 2: <domain> domain — tools <N>-<N+9> logged"
git push origin main
```

**3. Update the GitHub issue body** once per domain (not per tool):
After finishing all tools in a domain, append a domain summary block
to the issue body:
```
### <domain> (<X pass, Y fail, Z fishy, W skipped>)
<one line per Fail or Fishy tool with the error/symptom>
```

#### d) Findings — auto-draft, batch for review at end

Do NOT stop to ask about findings during the run. Instead:
- Keep a running list of all Fail and Fishy results in memory
- At the end of the entire run (after Pass B), print all finding drafts
  at once for review
- Format each draft the same way as Phase 1 (Symptom / Reproduction /
  Tool response / Possible cause / Recommendation)
- Wait for "file it / skip / merge with #X" decisions on each one

This is the only point where the run pauses for human input.

---

## Pass A — Read tool ordering

Run domains in this order (roughly: core first, exotic last):

1. `core` — remaining tools not in Phase 1
2. `incident` — remaining tools not in Phase 1
3. `change` — remaining tools not in Phase 1
4. `problem` — remaining tools not in Phase 1
5. `knowledge` — remaining tools not in Phase 1
6. `task` — remaining tools not in Phase 1
7. `catalog` — remaining tools not in Phase 1
8. `user` — remaining tools not in Phase 1
9. `updateset`
10. `sys-properties`
11. `reporting`
12. `performance`
13. `security`
14. `atf`
15. `agile`
16. `itam`
17. `integration`
18. `notification`
19. `flow`
20. `portal`
21. `workspace`
22. `app-studio`
23. `deployment`
24. `script`
25. `ml` (Table API tools only — skip Now Assist tools)
26. `va`
27. `mobile`
28. `hrsd` (expect most to skip — plugin likely absent)
29. `csm` (expect most to skip — plugin likely absent or NOW_ASSIST gated)
30. `devops` (expect most to skip — plugin likely absent)

---

## Pass B — Write tool ordering

Same domain order as Pass A. For each write tool:

- Use `[MCP-VALIDATION-PHASE2-<DATE>]` as the tag in short_description
  or name
- For update/patch tools: first create a record (or use one created
  earlier in Pass B), then update it
- For delete tools: only delete records created during this Pass B run,
  never pre-existing PDI data
- For script/deploy tools: use minimal safe payloads, no production
  impact expected on a PDI

---

## Wrap-up — after last Pass B tool

### Step 1 — Final commit

```bash
git add solvvision/runs/phase-2-run-${DATE}.md
git commit -m "Phase 2 run complete — <DATE>"
git push origin main
```

### Step 2 — Tally

Calculate across both passes:
- ✅ Pass / ❌ Fail / ⚠️ Fishy / ⏭️ Skipped counts
- Total tools tested
- Breakdown by domain

### Step 3 — Update runs index

Edit `solvvision/runs/README.md`, add a row to the Phase 2 table:

```markdown
| 1 | #<N> | dev229998 | <DATE> | <X pass / Y fail / Z fishy / W skipped> |
```

Commit and push.

### Step 4 — Print all finding drafts

Print every Fail and Fishy result as a finding draft (same format as
Phase 1). Wait for file / skip / merge decisions on each one.

### Step 5 — Cleanup list

Print all records created during Pass B with their sys_ids:

```
=== CLEANUP REQUIRED ===
All tagged [MCP-VALIDATION-PHASE2-<DATE>]:
- <table>: <number/name> (sys_id: ...)
...
Say "run cleanup" to delete via MCP, or delete manually in PDI UI.
```

### Step 6 — Final print

```
=== PHASE 2 COMPLETE ===

Validation run issue: <URL>
Run log: solvvision/runs/phase-2-run-<DATE>.md
Tools tested: <N> of ~320 in-scope
Final tally: <X pass / Y fail / Z fishy / W skipped>
Findings filed: #N, #N+1, ...
Cleanup outstanding: yes/no

Combined Phase 1 + Phase 2 coverage:
  Tested: <N> tools
  Excluded (Now Assist): 15
  Excluded (stubs/not-implemented): 3
  Total: 365
```

---

## Edge cases

### If a list tool returns empty unexpectedly
Classify as Fishy. Note "empty result — seed data expected on fresh PDI".
Continue — do not stop.

### If a get tool has no sys_id to work with
If the corresponding list tool also returned empty (Fishy), classify the
get tool as Skipped with note "no records available to fetch". Continue.

### If a tool hits an unexpected flag gate (like Phase 1 NOW_ASSIST issue)
Classify as Fail. Add to findings batch. Continue — do not stop.

### If MCP server connection drops mid-run
Restart server, verify instance, resume from the last unlogged tool
(check run log to find it). Note the interruption in the run log.

### If the GitHub issue body approaches 65,536 chars
Switch to posting domain summaries as comments on the issue instead
of editing the body. Note this in the run log.

---

## Save this file

Save this brief at `solvvision/phase-2-execution-brief.md` and commit
it before starting the run.

Begin with Setup Step 1. No confirmation needed — proceed automatically
through all steps and all tools until the finding drafts review at the end.
