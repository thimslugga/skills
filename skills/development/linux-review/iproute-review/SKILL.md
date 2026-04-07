---
name: iproute-review
description: >
  Deep regression analysis of iproute2 patches. Use when reviewing commits to
  ip, tc, bridge, or other iproute2 utilities for regressions in JSON output,
  argument parsing, netlink handling, and kernel compatibility. Activates in
  iproute2 source trees.
license: MIT
compatibility: Requires git.
metadata:
  author: review-prompts
  version: "1.0"
---

## Activation

Detected by presence of `ip/`, `tc/`, `bridge/` directories, or
`include/libnetlink.h`.

## Workflow

### Step 1: Load Context

1. Load `references/review-core.md` — core review checklist
2. Load subsystem files based on what changed:
   - `references/json-output.md` — for JSON output changes
   - `references/argument-parsing.md` — for CLI parsing changes
   - `references/kernel-compat.md` — for new kernel feature support
   - `references/coding-style.md` — for style questions
   - `references/netlink.md` — for netlink attribute handling

### Step 2: Review

Analyze the top commit (or provided patch/commit) against the checklist.

### Step 3: Report

If regressions found, create `review-inline.txt` in email format suitable
for the netdev mailing list.

## Key iproute2 Rules

- **matches() vs strcmp()**: new code MUST use `strcmp()`, not `matches()`
- **JSON output**: all display output MUST use `print_XXX()` helpers
- **Error handling**: stderr for errors, proper cleanup on failures
- **Netlink**: proper attribute handling and error checking
- No "Christmas tree" variable ordering required (this is userspace)
- Error messages must go to stderr to preserve JSON output

## Key Differences from Linux Kernel

iproute2 is userspace code:
1. No "Christmas tree" variable declaration ordering
2. New argument parsing must use `strcmp()`, not `matches()`
3. All output must use JSON-aware `print_XXX()` helpers
4. Error messages to stderr to preserve JSON output
5. No kernel docbook documentation format
