# Validation plan

## Why we validate

The upstream fork ([aartiq/servicenow-mcp](https://github.com/aartiq/servicenow-mcp))
exposes 365 tools across 31 domains. Code-level inspection has surfaced 12
tools with stub, not-implemented, or misleading-name issues — see
[stubs-and-caveats.md](stubs-and-caveats.md).

Before relying on this server for Solvvision consulting work, we validate
that the remaining tools actually behave as advertised when invoked against
a real ServiceNow instance. The validation has two phases.

## Phase 1 — manual checklist

**Approach:** A human runs prompts in Claude Code against a configured
ServiceNow PDI, reads the output, and records the result.

**Coverage:** 30 representative tools, one per working domain (the
`now-assist` domain is excluded for Phase 1 — its tools depend on plugin
entitlements that free PDIs may not have).

**Format:** Six themed mini-stories that simulate consultant workflows,
rather than a flat list of tool calls. This makes prompts feel natural to
Claude Code's LLM, exercises tool chaining, and produces a more useful
artefact for the rollout note.

**Mix:** 25 read operations, 5 write operations.

**Test record tagging:** every record created during validation is
prefixed `[MCP-VALIDATION-<DATE>]` in its description so it can be
filtered and cleaned up after.

**Artefact:** Each run is a GitHub Issue using the
[Validation run template](../.github/ISSUE_TEMPLATE/validation-run.md).

See [phase-1-checklist.md](phase-1-checklist.md) for the full prompt list.

## Phase 2 — scripted harness (deferred)

**Approach:** A small program spawns the MCP server as a subprocess and
talks JSON-RPC over stdio — the same protocol Claude Code uses. It calls
tools with pre-written arguments, records every response, produces a
pass/fail report.

**Coverage:** ~50 additional tools beyond Phase 1's 30. Total coverage
~80 of 365 tools (~22%).

**Why not start with Phase 2:** Manual validation is faster to set up,
removes ambiguity about *what* a passing response looks like (a human
calibrates that intuition first), and produces an artefact that doubles
as a Solvvision rollout demo. The scripted harness is most valuable for
regression testing once the fork starts being extended — that's a later
problem.

**Decision on Phase 2 scope:** Made after Phase 1 results.

## Result classification

| Marker | Meaning |
|---|---|
| ✅ Pass | Tool returned real ServiceNow data with the expected shape |
| ❌ Fail | Tool errored, threw, or returned an HTTP error response |
| ⚠️ Fishy | Tool returned successful-looking but suspicious output |
| ⏭️ Skipped | Pre-condition not met (plugin missing, write flag off, etc.) |

⏭️ Skipped is **not** a failure — it's a finding about the target
instance's plugin configuration, which is its own useful piece of data.

## Target instances

| Alias | Instance | Notes |
|---|---|---|
| dev229998 | dev229998.service-now.com | Solvvision-personal PDI, basic auth |

(Add rows as more instances are validated.)

## Out of scope for Phase 1

- The 12 tools documented in [stubs-and-caveats.md](stubs-and-caveats.md)
  (3 are pure stubs / not implemented, 7 are hard-coded table aliases that
  don't add capability beyond `query_records`, 2 have misleading names).
- The 15 Now Assist tools (require plugin entitlement; out of Phase 1).
- Tools that require explicit instance configuration we haven't done yet
  (some integration and custom-app tools).
