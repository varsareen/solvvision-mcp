# Solvvision MCP — Fix Waves

This folder tracks all fixes made to the Solvvision fork of aartiq/servicenow-mcp,
organised into three waves by complexity. All fixes stay in this fork (not raised upstream).

## Waves

| Wave | Branch | Complexity | Description |
|---|---|---|---|
| 1 | wave-1-low-complexity-fixes | Low | Spurious NOW_ASSIST_ENABLED guard removals on plain Table API tools (Category B) and wrong table name fixes |
| 2 | wave-2-medium-complexity-fixes | Medium | Broken round-trips, misleading names, wrong endpoints — need investigation before fixing |
| 3 | wave-3-high-complexity-fixes | High | Workflow gate issues — tools that lie about success when ServiceNow Business Rules block state transitions |

## How fixes are tracked

Each wave has a `fix-log.md` that records every fix made: which finding it closes,
which files were changed, and what the fix was. Claude Code appends a row to the
relevant fix-log after every fix it makes.

## Finding issues

All findings are tracked in `solvvision/solvvision-mcp-findings.xlsx` and as
GitHub issues #3, #5–#12.

## Wave status

| Wave | Status |
|---|---|
| Wave 1 | Not started |
| Wave 2 | Not started |
| Wave 3 | Not started |
