# Solvvision additions

This folder contains internal validation, test planning, and gap documentation
maintained by Solvvision (a ServiceNow consulting firm) on top of the
[aartiq/servicenow-mcp](https://github.com/aartiq/servicenow-mcp) upstream.

Nothing in this folder modifies upstream code. It exists alongside upstream
as a record of how this fork is being used and validated for Solvvision-internal
ServiceNow consulting work.

## Why this folder exists

The upstream fork exposes 365 tools across 31 domains, designed to let LLM
clients (e.g. Claude Code) interact with a ServiceNow instance over MCP. Before
relying on this server for consulting work, we systematically validate that
the tools behave as advertised against real ServiceNow instances. This folder
holds that validation work.

## Contents

| File | Purpose |
|---|---|
| [VALIDATION.md](VALIDATION.md) | Overall validation plan and methodology |
| [phase-1-checklist.md](phase-1-checklist.md) | The 30-prompt test checklist used in Phase 1 runs |
| [stubs-and-caveats.md](stubs-and-caveats.md) | Documented gaps — tools that don't fully do what their name implies |
| [runs/README.md](runs/README.md) | Index of past validation run issues |

## How to find past validation runs

Validation runs are tracked as GitHub Issues with the `validation` label.
Each run uses the [Validation run issue template](../.github/ISSUE_TEMPLATE/validation-run.md).

- [All validation issues](../../issues?q=label%3Avalidation)
- [Open validation issues](../../issues?q=is%3Aopen+label%3Avalidation)
- [runs/README.md](runs/README.md) — manually-maintained index

## Relationship to upstream

This is a fork. Validation findings are not pushed upstream automatically.
When findings are general-purpose (i.e. apply to anyone using upstream, not
just Solvvision-specific use cases), they may be raised as issues on the
upstream repository.

## Running a validation

See [VALIDATION.md](VALIDATION.md) for the full plan. The short version:

1. Configure your `.env` against a target ServiceNow PDI
2. Restart Claude Code so it picks up the new MCP server config
3. Open a new GitHub issue using the *Validation run* template
4. Work through the 6 mini-stories, ticking checkboxes as you go
5. Add comments for any findings, close the issue when done
6. Update `runs/README.md` with the issue link

## Contact

Solvvision-internal — see internal rollout notes for context.
