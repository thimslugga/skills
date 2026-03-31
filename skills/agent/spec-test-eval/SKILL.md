---
name: spec-test-eval
description: A structured framework for evaluating LLM and agent outputs using three layers: specs (what correct looks like), test functions (pass/fail checks), and evaluation functions (quality scoring).
triggers:
  - When you need to assess, grade, judge, or critique the quality of LLM or agent outputs
  - When designing evaluation criteria or building rubrics for prompt or model testing
  - When reviewing or comparing behavior or outputs from one or more agents or models
use_cases:
  - "How do I know if this output is good?"
  - "Evaluate this response"
  - "Grade this"
  - "Score this output"
  - "How should I measure quality?"
---

# Spec-Test-Eval Framework

A disciplined approach to evaluating LLM/agent outputs. The core idea: you cannot meaningfully evaluate anything without first defining what "good" means. This framework enforces that discipline through three layers, applied in order.

## The Three Layers

### Layer 1: Spec (What does "correct" look like?)

The spec is the upstream source of truth. Without it, tests check arbitrary things and scores measure nothing meaningful.

Before evaluating any output, define the spec by answering:

- **Task definition**: What was the LLM/agent asked to do? Restate it precisely -- vague tasks produce vague evaluations.
- **Success criteria**: What properties must a good output have? Be concrete. "Good summary" is useless. "Captures the three main arguments, stays under 200 words, does not introduce claims absent from the source" is a spec.
- **Boundary conditions**: What should the output explicitly *not* do? Constraints are part of the spec. If the output should not hallucinate facts, say so. If it should not include code, say so.
- **Format requirements**: Expected structure, length, tone, or style if relevant.

Write specs as plain declarative statements. Each statement should be independently verifiable.

**Example spec for a "summarize this article" task:**

```
1. Output is 2-4 sentences.
2. Captures the article's primary claim.
3. Mentions at least one piece of supporting evidence from the article.
4. Does not introduce information absent from the source.
5. Uses neutral tone (no editorializing).
```

**Common failure mode**: Skipping the spec and jumping straight to "does this look good?" That is vibes-based evaluation. It is unreliable, inconsistent, and impossible to improve systematically because there is nothing written down to revise.

### Layer 2: Test Functions (Pass or fail?)

Test functions are binary checks derived from the spec. Each spec statement becomes one or more tests. A test takes the output (and optionally the input/context) and returns pass or fail.

For each spec statement, ask: "Can I check this with a yes/no question?"

From the summary spec above:

```
Test 1: Is the output 2-4 sentences?             -> pass/fail
Test 2: Does it state the article's primary claim? -> pass/fail
Test 3: Does it cite supporting evidence?          -> pass/fail
Test 4: Does it contain only source-grounded info? -> pass/fail
Test 5: Is the tone neutral?                       -> pass/fail
```

Tests can be:

- **Mechanical**: Checkable by simple rules (word count, format, presence of required sections). These are cheap and deterministic.
- **Judgmental**: Require interpretation (factual accuracy, tone, coherence). These are more expensive and may themselves need an LLM to assess, but they are still binary -- the output either meets the criterion or it does not.

**Aggregation**: Report the pass rate across all tests. An output that passes 5/5 tests meets the spec. An output that passes 3/5 partially meets it, and you know exactly which criteria it failed.

**Common failure mode**: Writing tests that do not trace back to a spec statement. If a test exists, it should be because the spec requires it. Orphan tests create confusion about what you are actually measuring.

### Layer 3: Evaluation Functions (How good is it?)

Evaluation functions return a score on a scale, capturing quality gradients that binary tests miss. Use them when "pass" is not granular enough -- when you need to rank outputs, track improvement over time, or optimize.

An eval function scores along a dimension. Dimensions come from the spec but measure degree rather than compliance:

```
Dimension: Faithfulness to source (0-5)
  0 = Entirely fabricated
  1 = Mostly fabricated, one or two accurate details
  2 = Mix of accurate and fabricated claims
  3 = Mostly accurate, minor unsupported inferences
  4 = Accurate, with trivial imprecision
  5 = Fully faithful, every claim traceable to the source

Dimension: Conciseness (0-3)
  0 = Severely over or under length, misses the point
  1 = Roughly right length but padded or omits key info
  2 = Appropriate length, minor fluff
  3 = Tight, nothing wasted, nothing missing
```

**Anchoring is critical.** A scale without anchor descriptions is just a number. Define what each score level means in concrete terms. Without anchors, a "3 out of 5" from one evaluation and a "3 out of 5" from another are not comparable.

**Composite scores**: You can combine dimension scores into an overall score. Weight dimensions by importance. A weighted sum is fine; do not overcomplicate it.

**Common failure mode**: Using eval functions without first having tests. If an output fails basic spec compliance (hallucinates facts, wrong format), a nuanced quality score is meaningless. Tests are the gate; eval functions are the gradient beyond the gate.

## Applying the Framework

### Step-by-step process

1. **Write the spec first.** Even a rough spec is better than none. You can refine it later.
2. **Derive tests from the spec.** One test per spec statement minimum. If you cannot write a test for a spec statement, the statement is too vague -- rewrite it.
3. **Run tests as a first pass.** Report pass/fail. If critical tests fail, stop here -- the output does not meet minimum requirements and a nuanced score adds nothing.
4. **Apply eval functions for outputs that pass the gate.** Score along each dimension. Report dimension scores individually *and* as a composite if useful.
5. **Cite specific evidence.** For every test result or score, point to the specific part of the output that justifies the judgment. "Fails Test 4" is incomplete. "Fails Test 4: the second sentence claims 'revenues doubled' but the source article states 'revenues grew 40%'" is actionable.

### When comparing multiple outputs

Use the same spec, tests, and eval functions across all candidates. This is the whole point of the framework -- it forces apples-to-apples comparison.

1. Run all outputs through the test battery first. Eliminate any that fail critical tests.
2. Score survivors with eval functions.
3. Rank by composite score, but report dimension breakdowns so you can see *where* outputs differ.

### When iterating on a prompt or agent

The framework doubles as a feedback loop:

1. Define spec for the task.
2. Run the agent, evaluate the output.
3. Identify which tests failed or which eval dimensions scored low.
4. Adjust the prompt/agent to address those specific gaps.
5. Re-evaluate. Compare scores across iterations to confirm improvement.

This is how you avoid blind prompt tweaking. Each iteration targets a specific deficiency identified by the framework, and the framework measures whether the fix worked.

### Calibration

If multiple people (or LLM calls) are applying the same eval function, calibrate first:

- Pick 3-5 example outputs spanning the quality range.
- Have each evaluator score them independently.
- Compare scores and discuss disagreements.
- Refine anchor descriptions until scores converge.

This matters especially when using an LLM as a judge -- if the scoring prompt is ambiguous, the LLM's scores will drift across runs.

## Quick Reference

```
Spec    = "What should it do?"       -> Declarative statements
Test    = "Did it do that?"          -> Pass/fail per statement
Eval    = "How well did it do that?" -> Score per dimension

Order matters: Spec first. Tests gate. Eval scores what passes.
No spec = no meaningful evaluation.
No tests = no minimum bar.
No anchored scale = no reliable scoring.
```

## Anti-Patterns to Avoid

- **Vibes-based evaluation**: "This looks pretty good" without criteria. Unreliable, inconsistent, unimprovable.
- **Score without spec**: Assigning a number without defining what the number means.
- **Orphan tests**: Tests that check things the spec never required.
- **Unanchored scales**: "Rate 1-5" without defining what each level means.
- **Skipping tests and going straight to scoring**: If the output hallucinates or is in the wrong format, a quality score is noise.
- **Moving the goalposts**: Changing the spec after seeing the output to justify a preferred result. Write the spec before you see the output.
- **Over-engineering**: Five spec statements and five tests is often plenty. Do not build a 50-dimension rubric for a simple task. Match the rigor to the stakes.
