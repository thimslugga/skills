# Yoda Mode Agent Skill

## Overview

A Claude Code skill that makes Claude respond in Yoda's speech pattern.
Short, inverted sentences. No filler. Code and technical terms stay exact.

## Install

```bash
claude install-skill ./yoda-mode.skill
```

Or copy manually:

```bash
mkdir -p .claude/skills
cp -a yoda-mode/skills/yoda-mode .claude/skills/yoda-mode
```

## Activate

In any Claude Code session:

- `/yoda`
- "yoda mode"
- "talk like yoda"
- "speak like yoda"

## Deactivate

- "normal mode"
- "stop yoda"

## What it does

- Inverts sentence structure (object/predicate before subject)
- Keeps responses to 2-4 sentences
- Kills preamble, filler, and hedging
- Avoids contractions ("do not" over "don't")
- Drops in occasional Yoda-isms ("Hmm.", "Much to learn, you have.")

## What it preserves

- Code blocks: written normally, full and runnable
- Technical terms: exact (`useEffect` stays `useEffect`)
- Error messages: verbatim
- File paths, commands, flags: unchanged
- Git commits and PR descriptions: normal professional English

## Example

**Normal Claude:**

```text
Sure! The issue you're experiencing is most likely caused by your
authentication middleware not properly validating the token expiry.
I'd recommend changing the less-than operator to less-than-or-equal.
```

**Yoda mode:**

```text
In auth middleware, the bug is. Token expiry comparison uses `<`
instead of `<=`. Change it on line 42, you must.
```

## What is the .skill file?

A gzipped tarball. Nothing special:

```bash
file yoda-mode.skill
# gzip compressed data

tar -tzf yoda-mode.skill
# yoda-mode/SKILL.md
```

## License

MIT
