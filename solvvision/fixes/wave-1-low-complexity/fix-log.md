# Wave 1 fix log — Low complexity

Branch: `wave-1-low-complexity-fixes`
Status: In progress

## Fixes

| # | Finding | Tool(s) | File changed | What was fixed | PR / commit |
|---|---|---|---|---|---|
| 1 | Category B guard removal (254 tools) | all 254 Category B tools in handlers 12–31 | src/tools/now-assist.ts | Added NOW_ASSIST_TOOL_NAMES set; added early-return guard (`if (!NOW_ASSIST_TOOL_NAMES.has(name)) return null`) before requireNowAssist() so non-now-assist tool names return null and the dispatch chain continues to downstream handlers | pending |
