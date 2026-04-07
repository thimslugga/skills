---
name: systemd-review
description: >
  Deep regression analysis of systemd patches and commits. Use when reviewing
  systemd code changes for regressions, memory safety issues, D-Bus protocol
  errors, and cleanup/ownership bugs. Activates in systemd source trees.
license: MIT
compatibility: Requires git. Works best with semcode MCP for code navigation.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

### Step 1: Load Core Context (MANDATORY)

1. Load `references/technical-patterns.md` — always load first
2. Load `references/review-core.md` — the complete review protocol
3. Load subsystem files based on what changed:

| Subsystem | Triggers | File |
|-----------|----------|------|
| Service Manager | src/core/, Unit, Manager, Job | `references/core.md` |
| Namespaces | namespace, unshare, setns, CLONE_NEW* | `references/namespace.md` |
| Containers | src/nspawn/, container, pivot_root | `references/nspawn.md` |
| D-Bus | sd-bus, dbus, bus_ | `references/dbus.md` |
| Cleanup | _cleanup_, TAKE_PTR, TAKE_FD | `references/cleanup.md` |

### Step 2: Analyze

1. Understand the commit's purpose
2. Identify all changed files and functions
3. Analyze for regressions following the protocol
4. If no commit specified, analyze HEAD

### Step 3: Report

If regressions found, create `review-inline.txt` — plain text, suitable for
GitHub PRs or mailing lists.

Always conclude with:
```
FINAL REGRESSIONS FOUND: <number>
```

## Key systemd Conventions

- Return negative errno: `return -EINVAL;`
- Use `RET_NERRNO()` for libc calls
- Library code doesn't log (except DEBUG level)
- Use `_cleanup_*` attributes extensively
- Use `TAKE_PTR()`/`TAKE_FD()` for ownership transfer
- Initialize cleanup variables to NULL/-EBADF
- ALL FDs must have O_CLOEXEC from creation
- Use `safe_close()` which handles -EBADF
- No threads in PID1, no NSS calls from PID1

## Semcode Integration

When available, use semcode MCP tools:
- `find_function`/`find_type`: get definitions
- `find_callchain`: trace call relationships
- `find_callers`/`find_calls`: explore call graphs
- `grep_functions`: search function bodies
- `diff_functions`: identify changed functions in patches
