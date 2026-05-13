# Brief for Claude Code — Phase 1 validation execution

## Context

You are continuing the same session that committed the validation framework
(commit `7aa8200` — "Add Solvvision validation framework"). The repo is at
`varsareen/solvvision-mcp`, working dir
`~/Library/CloudStorage/OneDrive-SOLVVision/Solvvision Projects/claude-code-rollout/solvvision-mcp`,
issue template registered, `.env` configured for `dev229998.service-now.com`,
MCP server known to start with 365 tools loaded.

This brief executes the **Phase 1 manual checklist** — 30 prompts across
6 mini-stories — against the live PDI. The full prompt content is in
`solvvision/phase-1-checklist.md` and the issue template at
`.github/ISSUE_TEMPLATE/validation-run.md` (read both before starting).

You will play **two roles**:

- **Tester** — invoke MCP tools, read responses, classify results.
- **Scribe** — update the GitHub issue body and run log as we go, and
  (with my approval) file finding issues for failures.

Default mode is **bypass-permissions** — but read the *Stop-points* section
below carefully. There are explicit human-confirmation gates that override
bypass mode.

---

## Stop-points (rules)

These override bypass-permissions. You **MUST** stop and wait for explicit
human input at each:

1. **Before filing any `finding` issue.** Draft the issue (title, body,
   labels), show it, wait for "file it" / "skip" / "merge with #N".
2. **At the end of each story** (S1, S2, S3, S4, S5, S6). Print a brief
   summary of the story's results and ask "continue to next story?"
   This lets the human pause and resume across multiple sessions.
3. **If any tool invocation hangs or errors in an unexpected way** that
   isn't covered by the prompt's "Watch for" notes — surface it before
   classifying.
4. **If the validation run issue body grows beyond GitHub's 65,536-char
   limit** — surface it. We'll switch to comments instead of body edits.

You do **NOT** need to stop before:
- Write/script tools (S1.1, S1.6, S2.1, S4.2, S6.5) — PDI is throwaway,
  test records are tagged `[MCP-VALIDATION-2026-05-04]` for cleanup.
- Pass results — just record and move on.
- Skipped results when the cause is a known plugin gap (CSM, HRSD, DevOps,
  Predictive Intelligence) — note and move on.
- The proactive logging step (section e below) — do it every time, no
  confirmation needed.

---

## Setup phase — do these in order before the first prompt

### Step 1 — Create the 4 labels

```bash
gh label create validation --color "0366d6" --description "Validation run issues" 2>/dev/null || echo "validation label already exists"
gh label create phase-1 --color "a2eeef" --description "Phase 1 — manual checklist runs" 2>/dev/null || echo "phase-1 label already exists"
gh label create phase-2 --color "0e8a16" --description "Phase 2 — scripted harness runs" 2>/dev/null || echo "phase-2 label already exists"
gh label create finding --color "d93f0b" --description "Documented gap discovered during validation" 2>/dev/null || echo "finding label already exists"
```

### Step 2 — Open the validation run issue

```bash
COMMIT_SHA=$(git rev-parse --short HEAD)
echo "Fork commit: $COMMIT_SHA"
```

Then create the issue. **Do not** use the template flag (`--template`) on
the CLI — that's web-UI-only. Instead, read the template content, do the
substitutions in your head (Target PDI = dev229998.service-now.com, Date =
2026-05-04, Phase = 1, Tester = Varun Sareen, Fork commit = the SHA), and
pass the populated body via `--body-file`:

```bash
cp .github/ISSUE_TEMPLATE/validation-run.md /tmp/run-body.md
# Edit /tmp/run-body.md to:
#   - Remove the YAML frontmatter (everything between --- lines at top)
#   - Fill in the metadata table (PDI, Date, Phase, Tester, Fork commit)
#   - Replace all <DATE> placeholders with 2026-05-04
#   - Leave all checkboxes UNCHECKED (we'll tick them as we go)

gh issue create \
  --title "Validation run — dev229998 — 2026-05-04" \
  --body-file /tmp/run-body.md \
  --label validation \
  --label phase-1
```

The command will print the issue URL. **Save the issue number** — you'll
need it for every body update during the run.

### Step 3 — Initialise the run log

Create `solvvision/runs/phase-1-run-2026-05-04.md` with this header:

```markdown
# Phase 1 run log — dev229998 — 2026-05-04

| Prompt | Tool | Classification | Notes |
|--------|------|----------------|-------|
```

Commit it:

```bash
git add solvvision/runs/phase-1-run-2026-05-04.md
git commit -m "Start Phase 1 run log — dev229998 — 2026-05-04"
git push origin main
```

### Step 4 — Confirm setup

Print:

```
=== SETUP COMPLETE ===

Labels created/verified: validation, phase-1, phase-2, finding
Validation run issue: <URL>
Issue number: #<N>
Fork commit: <SHA>
Run log: solvvision/runs/phase-1-run-2026-05-04.md

Ready to start Story 1. Confirm by saying "begin S1".
```

**Stop and wait for "begin S1".**

---

## Run loop — for each of the 30 prompts

For prompt `<S{n}.{m}>`:

### a) Read the prompt

Open `solvvision/phase-1-checklist.md` and find the section for
`<S{n}.{m}>`. Note the tool name, the prompt text, the "Watch for" notes,
and the mode (read/write/script).

### b) Invoke the tool

Invoke the tool the prompt targets, with arguments derived from the prompt
text. Use real values (the actual `[MCP-VALIDATION-2026-05-04]` prefix in
created records, the actual `glide.product.description` property name, etc.).

### c) Show the response

Print the tool's full response in chat. Don't paraphrase — the human
needs the raw output to verify your classification.

### d) Classify

Choose one:

- ✅ **Pass** — tool returned real ServiceNow data with the expected shape
- ❌ **Fail** — tool errored, threw, or returned an HTTP error response
- ⚠️ **Fishy** — tool returned successful-looking but suspicious output
  (e.g. empty when seed data should exist, hardcoded-looking values,
  fields missing)
- ⏭️ **Skipped** — pre-condition not met (plugin missing, write flag off,
  etc.). This is a finding about the instance, not a tool failure.

State the classification and one sentence of reasoning.

### e) Record the result — do this immediately, every time, no confirmation needed

Do not batch. After every single prompt result, before moving to the next,
do all four of the following:

**1. Append a row to the run log** (`solvvision/runs/phase-1-run-2026-05-04.md`):

```
| S{n}.{m} | <tool name> | <emoji> <classification> | <one-line summary — include record number/sys_id if a record was created> |
```

**2. Update the validation run issue body** (`/tmp/run-body.md`):
- Tick the right result checkbox for `<S{n}.{m}>`
- Fill in the "Notes:" line with the same one-line summary

**3. Push the run log commit:**

```bash
git add solvvision/runs/phase-1-run-2026-05-04.md
git commit -m "S{n}.{m}: <classification> — <tool name>"
git push origin main
```

**4. Push the issue body update:**

```bash
gh issue edit <N> --body-file /tmp/run-body.md
```

This means every result is on GitHub within seconds. If the session dies,
nothing is lost.

### f) Findings — only for Fail or Fishy

If the result is Fail or Fishy, draft a finding issue. Show:

```
=== FINDING DRAFT (NOT YET FILED) ===

Title: [proposed title]
Labels: finding[, others if relevant]
Body:
  ## Symptom
  [what happened]

  ## Reproduction
  Step S{n}.{m} of validation run #N. Prompt was: "..."

  ## Tool response
  ```
  [raw response]
  ```

  ## Possible cause
  [your best honest read]

  ## Recommendation
  [skip in tests / fix in fork / raise upstream / accept]

  Surfaced in: #<N>

What should I do? File it / skip / merge with #X?
```

**Stop and wait** for one of:
- `file it` → run `gh issue create` with the draft, then add the new issue
  number to the run log row and the validation run issue's "Notes:" line
- `skip` → don't file, but record the finding in the Notes line
- `merge with #X` → don't file new; instead `gh issue comment X --body
  "Also surfaced in step S{n}.{m} of run #<N>: [one-line context]"`

### g) Move to next prompt

Print a one-line transition:

```
[S{n}.{m} complete: <classification>. Moving to S{n}.{m+1}...]
```

Then start the next prompt.

---

## Story stop-points

After the **last** prompt of each story (S1.6, S2.4, S3.4, S4.4, S5.5,
S6.7), stop and print:

```
=== STORY {n} COMPLETE ===

S{n}.1 - <classification>: <one-line summary>
S{n}.2 - <classification>: <one-line summary>
...

Story tally: X pass, Y fail, Z fishy, W skipped
Findings filed this story: #M, #M+1
Findings skipped: 0
Findings merged: #X (S{n}.{m} added as comment)

Continue to Story {n+1}? (or stop and resume later)
```

**Wait for "continue" or "stop here, resume later".**

If the human says stop, save your run state by:

1. Pushing any pending body edits to the validation run issue
2. Pushing any pending run log commits
3. Adding a comment to the issue: `Paused after S{n} on 2026-05-04. Will
   resume at S{n+1}.`
4. Print: `=== PAUSED. Resume by saying "continue from S{n+1}". ===`

---

## Final wrap-up — after S6.7 is classified

### Wrap step 1 — Tally

Calculate:
- ✅ Pass count out of 30
- ❌ Fail count out of 30
- ⚠️ Fishy count out of 30
- ⏭️ Skipped count out of 30
- Total findings filed (separate issues)
- Total findings skipped (recorded but not filed)

### Wrap step 2 — Headline findings

Fill in the "Headline findings" section of the validation run issue:

- **S4.2 `execute_background_script`:** what actually happened, exact
  error message if any
- **PDI plugins missing:** comma-separated list (CSM, HRSD, DevOps, etc.)
- **Surprising or hardcoded-looking data:** any from Fishy results
- **Schema validation errors:** any encountered (e.g. urgency type coercion)
- **LLM tool selection issues:** did the MCP layer route prompts to the
  right tools, or did anything pick a different tool than expected?

### Wrap step 3 — Update runs index

Edit `solvvision/runs/README.md`. Replace the placeholder row in the
"Phase 1 runs" table with a real row:

```markdown
| 1 | #<N> | dev229998 | 2026-05-04 | <X pass / Y fail / Z fishy / W skipped> |
```

Commit and push:

```bash
git add solvvision/runs/README.md
git commit -m "Record Phase 1 run #<N> against dev229998 (2026-05-04)"
git push origin main
```

### Wrap step 4 — Cleanup pass

Print the cleanup checklist with the actual record numbers/sys_ids created
during the run:

```
=== CLEANUP REQUIRED ===

Records created during this run, all tagged [MCP-VALIDATION-2026-05-04]:
- Incident: <INC0010xxx> (sys_id: ...)
- Problem: <PRB000xxx> (sys_id: ...)
- Change: <CHG000xxx> (sys_id: ...)
- Agile story: <STRYxxx> (sys_id: ...)
- Background script artefacts (S4.2): [none expected, but check]

Delete these manually in the PDI UI, or run a cleanup script in this
session if you want me to do it.
```

### Wrap step 5 — Final print

```
=== PHASE 1 COMPLETE ===

Validation run issue: <URL>
Run log: solvvision/runs/phase-1-run-2026-05-04.md
Findings filed: #M, #M+1, ...
Cleanup outstanding: yes/no

Next decision points:
- Close the validation run issue (#<N>) once findings are dispositioned
- Decide on Phase 2 scope (~50 more tools via scripted harness)
- Optionally update solvvision/stubs-and-caveats.md with newly-discovered
  caveats from this run
```

**Do not auto-close the validation run issue.** That's a deliberate
human action.

---

## Edge cases

### If the MCP server isn't loaded in this session

Restart it:

```bash
node dist/server.js < /dev/null 2>&1 &
```

…and confirm 365 tools loaded before continuing.

### If `gh issue edit` rejects the body

GitHub limits issue bodies to 65,536 characters. The validation run body
will get long but shouldn't approach that. If it does, switch to posting
findings as comments on the run issue instead of inline edits, and note
this in the wrap-up.

### If a tool throws a transport-level error (not a ServiceNow error)

That's a bug in the MCP server, not in the tool. Classify as Fail, file
a finding labelled `finding` and (additionally) `bug`, and continue.

### If the same finding surfaces twice

E.g. multiple tools fail because Now Assist plugin isn't activated. File
the *first* one as a finding issue, then for subsequent occurrences use
"merge with #X" to add a comment to the existing issue rather than filing
duplicates.

### If resuming after a paused session

1. Read the current run log (`solvvision/runs/phase-1-run-2026-05-04.md`)
   to see which prompts are already done.
2. Read the current validation run issue body to get the issue number and
   current state.
3. Copy the issue body back to `/tmp/run-body.md` for continued editing:
   ```bash
   gh issue view <N> --json body -q .body > /tmp/run-body.md
   ```
4. Resume from the next unlogged prompt.

---

## End of brief

Save this file at `solvvision/phase-1-execution-brief.md` in the repo so
it survives session boundaries. When you finish Phase 1, the artefacts
that should exist are:

1. The validation run issue (open or closed) with all 30 results filled in
2. `solvvision/runs/phase-1-run-2026-05-04.md` — complete run log
3. N finding issues (where N depends on what surfaced)
4. An updated `solvvision/runs/README.md` with this run logged
5. Cleanup completed in the PDI

Begin with the Setup phase. Stop after Step 4 ("begin S1") and wait.
