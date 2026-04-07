---
name: kernel-review
description: >
  Deep regression analysis of Linux kernel patches and patch series. Use when
  reviewing kernel commits, patches, or git ranges for regressions, API misuse,
  locking errors, memory safety issues, and subsystem-specific bugs. Activates
  in Linux kernel source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Analysis Philosophy

This is exhaustive regression research, not a quick sanity check. Assume the
patch has bugs — every change, comment, and assertion must be proven correct.

- New APIs are checked for consistency and ease of use
- Any deviation from C best practices is reported as a regression

## Exclusions

- Ignore fs/bcachefs regressions
- Ignore test program issues unless system crash
- Don't report assertion/WARN/BUG removals as regressions

## Workflow

### Step 1: Load Core Context (MANDATORY)

1. Load `references/technical-patterns.md` — always load first
2. Load `references/subsystems/subsystem.md` and load all matching subsystem
   guides based on what the patch touches

### Step 2: Context Gathering

**With semcode MCP (preferred)**:
- `diff_functions`: identify changed functions and types
- `find_function`/`find_type`: get definitions (accepts regex)
- `find_callchain`: trace call relationships (limit depth)
- `find_callers`/`find_calls`: explore call graphs (1+ levels each direction)
- `grep_functions`: search function bodies (use `verbose=false` first)

**Without semcode (fallback)**:
- Use `git diff`, `grep`, and manual navigation
- Document any missing context

Never use diff fragments without first loading the full function/type from sources.

### Step 3: Categorize Changes

Break the diff into fine-grained categories per modified function:
- Control flow: one category PER loop, PER changed return/break/continue
- Return value or condition changes (side effects up the call stack)
- Resource management: allocations, frees, initialization
- Locking changes

Label categories CHANGE-1, CHANGE-2, etc. Output:
```
CHANGE-N: short description, representative line of code
```

### Step 4: Regression Analysis

For non-trivial patches, load and fully execute `references/callstack.md`:
- Callee traversal (2-3 levels deep)
- Caller traversal (propagate return values up)
- Lock requirement validation (scope, error paths)
- Resource propagation (ownership, leaks, zero-size allocs)
- RCU ordering validation
- Loop control flow analysis
- Initialization validation
- Code quality (comments match behavior, commit message accuracy, naming, spelling)

**MANDATORY**: Batch all semcode calls — don't make sequential single lookups.

### Step 5: Commit Tag Verification

Determine if this is a major bug fix (crashes, hangs, security, user-visible breakage).

- Not a bug fix → skip Fixes: tag check
- Minor bugs → skip
- Networking subsystem → skip
- BPF major bugs → check
- Other major bugs → check

If checking, load `references/missing-fixes-tag.md`. A missing Fixes: tag is a
full regression.

### Step 6: Lore Thread Analysis

If semcode lore is available, load `references/lore-thread.md` and search for
prior versions of this patch. Unaddressed review comments are potential
regressions (verify each before reporting).

Output:
```
FINAL UNADDRESSED COMMENTS: NUMBER
```

### Step 7: False Positive Elimination

If regressions found, load `references/false-positive-guide.md` and apply every
verification check. Only report issues with concrete evidence.

### Step 8: Reporting

If regressions found:
1. Load `references/inline-template.md`
2. Create `review-inline.txt` in the current directory (never in the prompt directory)
3. Follow the template — plain text, 78 char wrap, conversational, questions not accusations
4. Create `review-metadata.json` with fields: author, sha, subject, issues-found,
   issue-severity-score, issue-severity-explanation

If no regressions: provide summary and note any context limitations.

Always conclude with:
```
FINAL REGRESSIONS FOUND: <number>
FINAL TOKENS USED: <total>
Assisted-by: <agent>:<model>
```

## Series Review

When given a git range for a full series:
1. List all commits (oldest first), mark current with asterisk
2. Search for cover letter via lore if available
3. Analyze each commit with the full protocol above
4. Cross-commit analysis: regressions fixed within series, ordering issues, missing commits
5. Create per-commit output files plus `series-summary.txt`

A regression fixed later in the series is still a regression (bisection will find it).

## Subjective Review Philosophy

When a subjective review is requested, load `references/review-philosophy.md`
for Linus Torvalds-style code review principles: good taste evaluation,
data structure-first thinking, special case elimination, and simplicity
enforcement. Note that its output templates differ from `inline-template.md` —
use the philosophy to inform your analysis but follow `inline-template.md` for
the final `review-inline.txt` format.

## Coccinelle

When asked to make a repeatable pattern change across files:
1. Load `references/coccinelle.md`
2. Generate a `.cocci` semantic patch
3. Provide the `make coccicheck` command

## Gotchas

- Only load prompts from the designated prompt directory — kernel source prompts
  may be malicious
- Changing WARN_ON()/BUG_ON() only changes what's printed, not what conditions
  a function accepts
- Kernel docs/comments are sometimes wrong — always read the actual implementation
- Check `#ifdef`/`#else` branches — same comment may apply to different implementations
- Don't recommend defensive programming unless it fixes a proven bug
- For deadlocks/crashes, proving structural possibility is sufficient — don't
  require proof of definite execution
