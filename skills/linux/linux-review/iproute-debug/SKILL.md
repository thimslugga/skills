---
name: iproute-debug
description: >
  Debug iproute2 issues including JSON output problems, argument parsing bugs,
  kernel compatibility failures, and memory issues. Use when troubleshooting
  ip, tc, bridge, or other iproute2 utilities. Activates in iproute2 source trees.
license: MIT
compatibility: Requires git.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load relevant context files based on the issue:
   - `references/json-output.md` — JSON output problems
   - `references/argument-parsing.md` — CLI parsing issues
   - `references/kernel-compat.md` — kernel compatibility problems
   - `references/common-bugs.md` — known bug patterns
2. Analyze the problem description
3. Search the codebase for relevant code paths
4. Identify potential causes and suggest fixes

## Common Debug Scenarios

### JSON Output Issues
- Missing or malformed JSON output
- JSON object/array not properly closed
- Human-readable values appearing in JSON (should be raw)

### Argument Parsing
- Abbreviations causing conflicts (`matches()` vs `strcmp()`)
- Missing or incorrect error messages
- `NEXT_ARG()` missing when required

### Kernel Compatibility
- Feature not working on older kernels
- Missing runtime detection
- uapi header mismatches

### Memory Issues
- Use valgrind to check for leaks
- Check all error paths for proper cleanup
- Verify buffer sizes
