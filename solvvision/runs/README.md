# Validation runs

All runs are tracked as GitHub Issues with the `validation` label.

- [All validation issues](../../../issues?q=label%3Avalidation)
- [Open validation issues](../../../issues?q=is%3Aopen+label%3Avalidation)

## Phase 1 runs

| Run | Issue | PDI | Date | Result |
|---|---|---|---|---|
| 1 | [#2](../../../issues/2) | dev229998 | 2026-05-04 | 9 pass / 0 fail / 1 fishy / 20 skipped (20 skips = NOW_ASSIST guard; finding #3 search_knowledge relevance) |

## Phase 2 runs

| Run | Issue | PDI | Date | Result |
|---|---|---|---|---|
| 1 | [#4](../../../issues/4) | dev229998 | 2026-05-04 | 67 pass / 12 fail / 4 fishy / 261 skipped (344 total; 220+ skips = NOW_ASSIST guard; findings #5–#12) |

## Wave 1 runs

Wave 1 guard testing re-ran all Category B tools (+ reporting.ts) after the NOW_ASSIST guard bug was fixed.

| Run | Branch | PDI | Date | Result |
|---|---|---|---|---|
| 1 | `wave-1-low-complexity-fixes` | dev229998 | 2026-05-06 | 216 pass / 13 fail / 5 fishy / 38 skipped — 272 tools; 38 skips = plugin absent (HRSD 16, Mobile 8, DevOps 6, Agent Workspace 3, SecOps-partial 3, PAD 2); findings #13–25 raised |

## Updating this file

After closing a validation run issue:

1. Add a row to the relevant phase's table.
2. Link to the issue (e.g. `#4`).
3. Note the PDI alias, run date, and a one-line result summary.

This file is a manually-maintained convenience index; the GitHub Issues
list is the source of truth.
