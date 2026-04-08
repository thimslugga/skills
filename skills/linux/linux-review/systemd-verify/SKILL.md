---
name: systemd-verify
description: >
  Verify systemd review findings against false positive patterns. Use after
  systemd-review or systemd-debug to validate that reported issues are real
  bugs with concrete evidence. Activates in systemd source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load `references/false-positive-guide.md`
2. Apply each verification check systematically

## Verification Steps

For each reported issue:

1. Trace the exact code path that triggers it
2. Verify the path is actually reachable
3. Check if validation happens elsewhere
4. Confirm this is production code, not test/debug
5. Provide concrete evidence (code snippets, call chains)

## Output

For each issue:
- **VERIFIED ISSUE**: if the bug is real
- **ELIMINATED**: if the issue is a false positive
