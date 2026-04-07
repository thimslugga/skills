---
name: iproute-verify
description: >
  Verify iproute2 patch correctness and completeness. Use to validate that a
  patch compiles cleanly, handles JSON output correctly, follows coding style,
  and has proper commit metadata. Activates in iproute2 source trees.
license: MIT
compatibility: Requires git.
metadata:
  author: review-prompts
  version: "1.0"
---

## Workflow

1. Load `references/review-core.md` — verification checklist
2. Load `references/false-positive-guide.md` — eliminate false positives
3. Check all items systematically

## Verification Checklist

### Code Quality
- [ ] Follows coding style (tabs, line length, braces)
- [ ] No compiler warnings expected
- [ ] Memory properly managed

### Functionality
- [ ] JSON output works correctly
- [ ] Text output works correctly
- [ ] Error cases handled properly
- [ ] Works with expected kernel versions

### Commit Quality
- [ ] Subject line format correct
- [ ] Description adequate
- [ ] Signed-off-by present
- [ ] uapi changes (if any) in separate patch

## Output

Report verification results with PASS/FAIL per item, details on failures,
and suggestions for fixes.
