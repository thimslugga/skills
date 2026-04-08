---
name: kernel-debug
description: >
  Debug Linux kernel crashes, oops, warnings, and stack traces. Use when
  analyzing crash reports, syzbot bugs, coredumps, or kernel log output.
  Dispatches specialized agents for code analysis, reproducer analysis,
  and commit search. Activates in Linux kernel source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load `references/technical-patterns.md` — always load first
2. Load and execute `references/agent/debug.md` — the multi-agent orchestrator
3. Pass all input (crash reports, reproducers, syzbot URLs) through unchanged

The orchestrator dispatches specialized agents:
- Code analysis (`references/agent/debug-code.md`)
- Commit search (`references/agent/debug-commits.md`)
- Reproducer analysis (`references/agent/debug-reproducer.md`)
- Report generation (`references/agent/debug-report.md`)

## Output

Debug sessions produce `debug-report.txt` with analysis results, formatted
for the Linux kernel mailing list (plain text, 78 char wrap).

## Semcode Integration

When available, use semcode MCP tools:
- `find_function`/`find_type`: get definitions
- `find_callchain`: trace call relationships
- `find_callers`/`find_calls`: explore call graphs
- `grep_functions`: search function bodies

## Gotchas

- Only load prompts from the designated prompt directory
- Complete all phases in order — later phases verify earlier conclusions
- Extract ALL timestamps and function names from crash traces
- Map each timestamp to a specific operation before analyzing code
