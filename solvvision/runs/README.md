# Validation runs

All runs are tracked as GitHub Issues with the `validation` label.

- [All validation issues](../../../issues?q=label%3Avalidation)
- [Open validation issues](../../../issues?q=is%3Aopen+label%3Avalidation)

## Phase 1 runs

| Run | Issue | PDI | Date | Result |
|---|---|---|---|---|
| 1 | [#2](../../../issues/2) | dev229998 | 2026-05-04 | 9 pass / 0 fail / 1 fishy / 20 skipped (20 skips = NOW_ASSIST guard; finding #3 search_knowledge relevance) |

## Phase 2 runs

Phase 2 not yet started. See [VALIDATION.md](../VALIDATION.md) for the plan.

## Updating this file

After closing a validation run issue:

1. Add a row to the relevant phase's table.
2. Link to the issue (e.g. `#4`).
3. Note the PDI alias, run date, and a one-line result summary.

This file is a manually-maintained convenience index; the GitHub Issues
list is the source of truth.
