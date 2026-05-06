# Brief for Claude Code — Wave 1 fixes execution

## Context

This brief fixes all **Category B — spurious NOW_ASSIST_ENABLED guard** issues
identified during Phase 1 and Phase 2 validation. These are plain Table API
tools in non-now-assist domains that incorrectly check `NOW_ASSIST_ENABLED`
before executing, causing them to fail when Now Assist is disabled.

Repo: `varsareen/solvvision-mcp`
Working dir: `~/Library/CloudStorage/OneDrive-SOLVVision/Solvvision Projects/claude-code-rollout/solvvision-mcp`
Branch: `wave-1-low-complexity-fixes`
Mode: **fully unattended, --dangerously-skip-permissions**

Read `solvvision/stubs-and-caveats.md` and `solvvision/solvvision-mcp-findings.xlsx`
before starting to understand the full scope of fixes.

---

## What Category B means

Category B tools are tools **outside** `src/tools/now-assist.ts` that call
`requireNowAssist()` or check `NOW_ASSIST_ENABLED` at the top of their handler
before making a plain Table API call. The guard is spurious — these tools do
not use any Now Assist / AI capability. Removing the guard allows them to work
with `NOW_ASSIST_ENABLED=false`.

Category A tools (in `src/tools/now-assist.ts`) are **not touched in this wave**.
They are documented as unsupported in v1 — see the Excel update step below.

---

## Setup

### Step 1 — Switch to wave branch

```bash
git checkout wave-1-low-complexity-fixes
git pull origin wave-1-low-complexity-fixes
```

### Step 2 — Identify all Category B tools

Scan every file in `src/tools/` **except** `now-assist.ts` for:
- Any call to `requireNowAssist()` or equivalent
- Any check of `NOW_ASSIST_ENABLED` env var
- Any import of Now Assist utilities used only as a guard

Build a list of: filename → tool name → line number → guard expression.
Print this list before making any changes so it is visible for verification.

### Step 3 — Confirm instance is live

Call `get_current_instance` — should return `dev229998.service-now.com`.

Print:
```
=== WAVE 1 SETUP COMPLETE ===
Branch: wave-1-low-complexity-fixes
Category B tools identified: <N>
PDI: dev229998.service-now.com
Starting fixes...
```

Then proceed automatically with no further confirmation needed.

---

## Fix loop — for each Category B tool

### a) Apply the fix

Remove or comment out the spurious `requireNowAssist()` / `NOW_ASSIST_ENABLED`
guard from the handler. Do not change anything else in the handler — no
refactoring, no other edits.

If the guard is shared with legitimate Now Assist logic in the same handler
(i.e. the tool has both a plain Table API path AND a Now Assist path), only
remove the guard from the plain path. Flag this as a complex case and note it
in the fix log.

### b) Build

After every 10 fixes (or after finishing a domain file), run:

```bash
npm run build 2>&1
```

If the build fails, revert the last fix, note it in the fix log as
"build failure — reverted", and continue with the next tool. Do not
stop the wave.

### c) Append to fix log

Append a row to `solvvision/fixes/wave-1-low-complexity/fix-log.md`:

```
| <#> | Category B guard removal | <tool_name> | src/tools/<file>.ts | Removed requireNowAssist() guard on line <N> | pending |
```

### d) Commit every 10 fixes

```bash
git add src/tools/
git add solvvision/fixes/wave-1-low-complexity/fix-log.md
git commit -m "Wave 1: remove spurious NOW_ASSIST guard — <domain> domain (<N> tools)"
git push origin wave-1-low-complexity-fixes
```

---

## Wrong table name fixes (also Wave 1)

After all Category B guard removals are done, apply the wrong table name fixes
from the findings. These are findings #7, #8, #9, #10:

| Finding | Tool(s) | Current wrong table | Correct table |
|---|---|---|---|
| #7 | `create_approval_rule` | `sysapproval_rule` | `sysrule_approvals` — verify in PDI first |
| #8 | `list_atf_test_results` | `sys_atf_result` | `sys_atf_test_result` — verify in PDI first |
| #8 | `get_atf_failure_insight` | `sys_atf_failure_insight` | verify in PDI by querying sys_db_object |
| #9 | `create_scheduled_report` | `sys_report_schedule` | `sysauto_report` — verify in PDI first |
| #10 | `list_job_run_history` | `sysauto_trigger_log` | `sysauto_trigger` — verify in PDI first |

**For each wrong table fix:**

1. Verify the correct table name first by calling `query_records` with the
   proposed correct table name and limit=1. If it returns data or an empty
   array (not an INVALID_REQUEST), the table exists — proceed with the fix.
2. Apply the fix in the source file.
3. Build to confirm no errors.
4. Append to fix log with the verified correct table name.
5. Commit.

---

## Testing — Category B tools only

After all fixes are applied and the build passes, re-run every Category B
tool that was previously skipped due to the NOW_ASSIST_ENABLED guard.

For each tool:

### a) Invoke with minimal payload

Same approach as Phase 2 — no required args tools get `{}`, tools needing
a sys_id get one from a prior list call.

### b) Classify

- ✅ **Pass** — returns real data, no NOW_ASSIST error
- ❌ **Fail** — still returns NOW_ASSIST error (guard not fully removed) or
  new error introduced by the fix
- ⚠️ **Fishy** — returns 200 but suspicious output
- ⏭️ **Skipped** — different pre-condition blocking it (plugin absent etc.)

### c) Record result

Update the fix log row for this tool — replace "pending" in the PR/commit
column with the classification emoji and one-line note.

### d) If Fail — investigate immediately

If a tool still fails after the guard removal, check:
1. Is there a second guard elsewhere in the handler?
2. Did the build actually include this file's changes?
3. Is the tool in a domain with a shared guard utility?

Fix and retest before moving on. Do not leave a Fail unresolved.

### e) Commit test results

```bash
git add solvvision/fixes/wave-1-low-complexity/fix-log.md
git commit -m "Wave 1: test results logged for Category B tools"
git push origin wave-1-low-complexity-fixes
```

---

## Excel update — two changes

After testing is complete, update `solvvision/solvvision-mcp-findings.xlsx`:

### Change 1 — Update Category B rows in Findings sheet

For every Category B tool that now Passes:
- Set **Status** = Fixed
- Set **Notes** = "Spurious guard removed in wave-1-low-complexity-fixes branch"

For any that still Fail:
- Set **Status** = Open
- Set **Notes** = updated note explaining why it still fails

### Change 2 — Add Future Features sheet

Create a new sheet called **'Future Features'** with these columns:

| Issue # | Tool | Domain | Description | Why Deferred | Target Wave / Release |
|---|---|---|---|---|---|

Populate it with all Category A tools (genuine Now Assist tools from
`src/tools/now-assist.ts`) — one row per tool:
- Issue #: blank
- Description: the tool's description field from the source
- Why Deferred: "Requires Now Assist plugin — not enabled in Solvvision v1 deployment"
- Target Wave / Release: "v2 — post Now Assist enablement"

Commit and push the Excel file:

```bash
git add solvvision/solvvision-mcp-findings.xlsx
git commit -m "Wave 1: update findings tracker — Category B fixed, Category A moved to Future Features sheet"
git push origin wave-1-low-complexity-fixes
```

---

## Wrap-up

After fixes, tests, and Excel update are all done, print:

```
=== WAVE 1 COMPLETE ===

Branch: wave-1-low-complexity-fixes
Category B guard removals: <N> tools fixed
Wrong table name fixes: <N> tools fixed
Total fixes applied: <N>

Test results:
  ✅ Pass: <N>
  ❌ Fail: <N> (listed below if any)
  ⚠️  Fishy: <N>
  ⏭️  Skipped: <N> (different pre-condition)

Fix log: solvvision/fixes/wave-1-low-complexity/fix-log.md
Excel updated: solvvision/solvvision-mcp-findings.xlsx

Next step: review changes on GitHub, then merge wave-1-low-complexity-fixes into main.
To merge: gh pr create --title "Wave 1: low complexity fixes" --body "Closes findings from Phase 1/2 validation. See solvvision/fixes/wave-1-low-complexity/fix-log.md" --base main
```

**Do not merge to main automatically.** That is a manual step after you review.

---

## Edge cases

### If requireNowAssist() is defined in a shared utility used by both now-assist.ts and other domains
Only remove the call from the non-now-assist handler. Do not modify the
utility itself. Note this in the fix log.

### If a domain file has both Category A and Category B tools
Fix only the Category B tools in that file. Leave Category A tools untouched.

### If the build fails after a fix
Revert that specific fix with `git checkout src/tools/<file>.ts`, note it
in the fix log as "reverted — build failure", and continue.

### If a previously-passing tool breaks during testing
Stop, investigate, revert the relevant fix, and note it. Do not push
broken code.

---

## Save this file

This brief is saved at `solvvision/fixes/wave-1-low-complexity/wave-1-brief.md`.
Commit it before starting fixes.
