# Terse Mode Agent Skill

A Claude Code skill that makes Claude respond like a senior engineer
in a code review. Direct, precise and no filler. Proper grammar stays intact.

## Install

```bash
mkdir -p .claude/skills
cp -a terse-mode/skills/terse-mode .claude/skills/
```

## Activate

In any Claude Code session:

- `/terse`
- "terse mode"
- "be concise"
- "less tokens please"
- "short mode"
- "brief mode"

## Deactivate

- "normal mode"
- "verbose"
- "stop terse"

## What it does

- Starts with the answer. No "Sure!", "Great question!", "I'd be happy to help."
- No hedging. Drops "might", "perhaps", "it could be worth considering."
- No sign-off fluff. No "Let me know if you have questions!"
- Matches response length to question complexity. One-line questions get one-line answers.
- Leads with the fix, answer, or decision. Context comes after, only when needed.
- Sentence fragments where a full sentence adds nothing.

## What it preserves

- Code blocks: full, correct, runnable. Never truncated.
- Technical terms: exact names, never paraphrased.
- Error messages: verbatim.
- File paths, commands, flags: unchanged.
- Git commits, PR descriptions, docs: normal professional English.

## Example

**Normal Claude:**

```text
Sure! The issue you're running into is most likely related to the fact
that your middleware isn't correctly checking token expiration. What's
happening is that the comparison operator you're using doesn't account
for the edge case where the token expires at exactly the current time.
I'd recommend changing the less-than operator to a less-than-or-equal
operator. Here's how you can fix it:
```

**Terse:**

```text
Bug is in auth middleware. Token expiry comparison uses `<` instead
of `<=`, so tokens that expire at exactly `now` pass validation.

Fix: line 42 in auth.py, change `if token.exp < time.now()` to `<=`.
```

## What is the .skill file?

A gzipped tarball. Nothing special:

```bash
file terse.skill
# gzip compressed data

tar -tzf terse.skill
# terse/SKILL.md
```

## License

MIT
License

MIT
