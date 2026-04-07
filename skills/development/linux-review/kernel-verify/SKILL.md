---
name: kernel-verify
description: >
  Verify kernel patch review findings and eliminate false positives. Use after
  kernel-review or kernel-debug to validate that reported issues are real bugs
  with concrete evidence. Activates in Linux kernel source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load `references/technical-patterns.md` — always load first
2. Load `references/false-positive-guide.md`
3. Apply every verification check systematically

## Core Principle

**If you cannot prove an issue exists with concrete evidence, do not report it.**

For deadlocks, infinite waits, crashes, and data corruption, "concrete evidence"
means proving the code path is structurally possible — not proving it will
definitely execute on every run.

## Verification Checklist

For each reported issue:

1. Ensure full commit message / patch description is still in context
2. Check defensive programming requests — never suggest checks unless:
   - Input comes from untrusted source
   - An actual path exists where invalid data reaches the code
   - Current code can demonstrably fail
3. Check API misuse — prove an actual calling path triggers the issue
4. Verify error handling claims — prove the error is possible with the given arguments
5. Trace the exact code path that triggers the issue
6. Confirm the path is actually reachable
7. Check if validation happens elsewhere in the call chain
8. Confirm this is production code, not test/debug

## Output

For each issue:
- **VERIFIED ISSUE**: if the bug is real, with concrete evidence
- **ELIMINATED**: if the issue is a false positive, with explanation

## Gotchas

- Authors listed in MAINTAINERS: trust their comments and commit messages
  (but still verify unmodified code comments against implementation)
- "Caller should prevent this" is not sufficient to dismiss — prove the
  triggering condition is structurally impossible
- Load `references/pointer-guards.md` when analyzing NULL pointer issues
