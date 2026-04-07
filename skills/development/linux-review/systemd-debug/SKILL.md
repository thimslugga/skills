---
name: systemd-debug
description: >
  Debug systemd crashes, hangs, and unexpected behavior. Use when analyzing
  journalctl output, coredumps, stack traces, or error messages from systemd
  components. Activates in systemd source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load `references/technical-patterns.md` — always load first
2. Load `references/debugging.md` — the complete debugging protocol
3. Load subsystem files based on the affected component

## Expected Input

- journalctl output
- Coredump / stack trace
- Error messages
- Reproduction steps

## Output

Debug sessions produce `debug-report.txt` with analysis results, plain text
suitable for GitHub PRs or mailing lists.

## Key systemd Debugging Conventions

- Error codes are negative errno values
- `_cleanup_*` misuse causes use-after-free or double-free
- `TAKE_PTR()`/`TAKE_FD()` ownership transfer bugs are common
- PID1 has no threads — race conditions indicate fork/signal issues
- Check O_CLOEXEC on all FD creation paths
