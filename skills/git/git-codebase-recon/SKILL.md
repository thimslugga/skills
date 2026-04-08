---
name: git-codebase-recon
description: Git-based codebase reconnaissance and diagnostics. Use this skill whenever the user wants to understand a new codebase, audit a repo's health, figure out who knows what in a project, find problem areas in code, identify high-churn or bug-prone files, assess bus factor, check project velocity, or asks things like "where should I start reading this code", "what's the state of this repo", "who owns this code", "show me the hotspots", "is this project healthy", or "help me get oriented in this codebase". Also trigger when the user mentions codebase audits, legacy code exploration, onboarding onto a new project, or tech debt assessment via git history.
---

# Git Codebase Recon

When you land in a new codebase, the commit history tells you where the bodies are buried before you open a single file. These commands give you a diagnostic picture: who built it, where the problems cluster, whether the team ships with confidence or tiptoes around landmines.

Think of it like checking the medical chart before examining the patient.

## The Five Recon Commands

Run these first. They take a couple minutes and tell you where to focus your reading.

### 1. Churn Hotspots -- What Changes the Most

```bash
git log --format=format: --name-only --since="1 year ago" \
  | sort | uniq -c | sort -nr | head -20
```

The 20 most-changed files in the past year. The file at the top is almost always the one people warn you about.

**What to look for:**
- High churn alone is fine -- it might just be active development.
- High churn + nobody wants to own it = codebase drag. That's the file where every change is a patch on a patch and the blast radius of a small edit is unpredictable.
- Cross-reference the top 5 against the bug hotspot command below. A file that's high-churn AND high-bug is your single biggest risk.

**Tuning:** Adjust `--since` to match the project's age. For a 6-month-old repo, use `6 months ago`. For mature projects, `1 year ago` or `2 years ago` captures the relevant cycle.

### 2. Bus Factor -- Who Built This

```bash
git shortlog -sn --no-merges
```

Every contributor ranked by commit count. Quick reads:

- One person at 60%+ of commits = bus factor of 1.
- Compare against a recent window to see if they're still around:

```bash
git shortlog -sn --no-merges --since="6 months ago"
```

If the all-time top contributor doesn't show up in the last 6 months, the person who knows this code best is gone. That's worth flagging immediately.

**Also check the tail.** Thirty total contributors but only three active in the last year means the people who built it aren't the ones maintaining it.

**Caveat:** Squash-merge workflows compress authorship. If every PR gets squashed, the output shows who merged, not who wrote the code. Ask about the merge strategy before drawing conclusions.

### 3. Bug Clusters -- Where Do Bugs Live

```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' \
  | sort | uniq -c | sort -nr | head -20
```

Same shape as the churn command, but filtered to commits with bug-related keywords. Compare this list against churn hotspots -- files on both lists keep breaking and keep getting patched but never get properly fixed.

**Depends on commit discipline.** If the team writes "update stuff" for every commit, you'll get noise. But even a rough bug density map beats no map.

**Extra keywords** you might add depending on the project: `patch|workaround|hack|revert|crash`

### 4. Project Velocity -- Accelerating or Dying

```bash
git log --format='%ad' --date=format:'%Y-%m' | sort | uniq -c
```

Commit count per month for the full repo history. Scan the output for shapes:

| Pattern | What it usually means |
|---|---|
| Steady rhythm | Healthy continuous delivery |
| Sudden 50% drop in a single month | Someone left |
| Declining curve over 6-12 months | Team losing momentum or project winding down |
| Periodic spikes then quiet | Batch releases, not continuous shipping |

This is team data, not code data. It tells you about the humans behind the repo.

### 5. Firefighting Frequency -- How Often Do Things Blow Up

```bash
git log --oneline --since="1 year ago" \
  | grep -iE 'revert|hotfix|emergency|rollback'
```

Revert and hotfix frequency. Reading the output:

- A handful per year = normal
- Every couple of weeks = the team doesn't trust its deploy process (unreliable tests, missing staging, or painful rollbacks)
- Zero results = either very stable, or nobody writes descriptive commit messages

## Putting It Together

After running all five, you have a map:

1. **Start reading** at the high-churn, high-bug files. That's where the pain is.
2. **Talk to** the top contributors (if they're still around) about those files.
3. **Velocity + firefighting** tells you whether the team is shipping confidently or in survival mode.
4. **Bus factor** tells you the risk if someone leaves tomorrow.

## Bonus Commands

### File-level authorship for a specific problem file

```bash
git log --format='%an' -- path/to/problem-file.py | sort | uniq -c | sort -nr
```

Who has touched this file the most? That's who you ask questions.

### Recent activity on a specific directory

```bash
git log --oneline --since="3 months ago" -- src/payments/
```

Useful when scoping a specific area.

### Commit message word frequency (quick culture check)

```bash
git log --oneline --since="1 year ago" | awk '{$1=""; print}' \
  | tr ' ' '\n' | sort | uniq -c | sort -nr | head -30
```

If "WIP", "fix", and "temp" dominate, the team is moving fast and sloppy. If you see ticket IDs and conventional commit prefixes, there's discipline in place.

## Reference

- Source: [The Git Commands I Run Before Reading Any Code](https://piechowski.io/post/git-commands-before-reading-code/) by Ally Piechowski
- Related: [Microsoft Research - churn metrics predict defects better than complexity metrics](https://www.microsoft.com/en-us/research/publication/use-of-relative-code-churn-measures-to-predict-system-defect-density/) (2005)
