# spec-test-eval Agent Skill

The description is written to trigger broadly, so any time Claude needs to assess, grade, score, or critique LLM/agent output quality.

The body walks through the three layers in order: write the spec first (declarative statements of what correct looks like), derive binary tests from the spec (pass/fail gate), then apply eval functions with anchored scales (quality gradient for outputs that pass the gate). It also covers comparing multiple outputs, iterating on prompts/agents using the framework as a feedback loop, and calibration when multiple evaluators are involved.

The anti-patterns section at the end calls out the common traps, such as vibes-based evaluation, unanchored scales, skipping the spec, etc.

## Specifications (Specs)

A spec (specification) is a formal description of what something *should* do -- the expected behavior, written down before or alongside the implementation.

If an evaluation function asks "how good is this?" and a test function asks "does this pass?", a spec answers the prior question: **"pass according to what, exactly?"**

A spec is the source of truth that test functions are written against.

**Analogy:** Think of a spec like a blueprint for a building. The blueprint doesn't build anything itself, but every inspection (test) checks the actual construction against what the blueprint says should be there. Without the blueprint, the inspector has nothing to measure against.

In practice, specs show up at different levels of formality:

**Informal spec** -- a plain English description:

```
The login endpoint must return 401 if the API key is missing or invalid.
Passwords must be at least 12 characters with one uppercase and one digit.
```

**Executable spec** -- the spec *is* the test. This is what frameworks like RSpec popularized:

```ruby
describe "Password validation" do
  it "rejects passwords shorter than 12 characters" do
    expect(valid?("short")).to eq(false)
  end

  it "requires at least one uppercase letter" do
    expect(valid?("alllowercase1")).to eq(false)
  end
end
```

**Formal/machine-verifiable spec** -- mathematically precise. Think RFCs, protocol definitions, or something like TLA+. HTTP status codes, DNS record formats, TCP handshake behavior -- all defined by specs.

The relationship between the three concepts you've asked about flows naturally:

```
spec  -->  defines what "correct" means
test  -->  checks if something meets the spec (pass/fail)
eval  -->  scores how well something meets the spec (gradient)
```

A real-world example tying them together: say you're writing an Ansible role.

- The **spec** says "nginx must be installed, running, and listening on port 443."
- The **test function** checks: is nginx running? Is 443 open? Pass or fail.
- The **evaluation function** might score the overall hardening posture: nginx config quality, TLS version, cipher strength, headers present -- a score from 0-100.

The spec is always upstream. Without it, your tests are checking arbitrary things and your eval function is scoring against nothing meaningful. That's why "write the spec first" is such common advice -- it forces you to define what success looks like before you start building.

## Test Functions (Pass/Fail)

A test function is a function that checks whether something meets a specific condition and returns a pass/fail (boolean) result.

Where an evaluation function says "how good is this?" (a score on a spectrum), a test function says "does this pass or not?" (yes or no).

```python
# Evaluation function -- returns a score
def evaluate_password(pw):
    score = 0
    if len(pw) >= 12: score += 2
    if any(c.isupper() for c in pw): score += 1
    if any(c.isdigit() for c in pw): score += 1
    return score  # 0-4

# Test function -- returns pass/fail
def test_password(pw):
    return (len(pw) >= 12 and
            any(c.isupper() for c in pw) and
            any(c.isdigit() for c in pw))  # True/False
```

**Analogy:** An evaluation function is like a thermometer -- it gives you a reading. A test function is like a thermostat -- it just triggers on or off based on a threshold.

In practice, a test function is often just an evaluation function with a cutoff:

```python
def test_from_eval(state):
    return evaluate(state) >= some_threshold
```

Where you'll see test functions:

- **Unit testing:** `assert result == expected` -- the whole test either passes or fails.
- **Input validation:** Does this request have the required fields? Yes or no.
- **AI/ML evals:** Given a prompt and a model response, did the model get it right? Pass/fail. When people talk about "evals" for LLMs, they're often running a suite of test functions across many inputs.
- **Shell scripting:** `test -f /etc/config.yaml && echo "exists"` -- the `test` builtin itself is literally a test function.
- **Monitoring/alerting:** Is disk usage over 90%? Fire the alert or don't.

The key distinction is just output type. Evaluation functions give you a gradient you can optimize against. Test functions give you a binary gate you can branch on. Most systems use both -- evaluate to rank or optimize, then test to make final go/no-go decisions.

## Evaluation Functions (Quality Scoring)

An evaluation function is a function that takes some state or position and returns a numeric score representing how "good" or "bad" that state is.

The classic example is chess. A chess engine can't search every possible move all the way to checkmate -- there are too many. So at some point it stops looking ahead and asks: "How good is this board position right now?" That's what the evaluation function answers.

A dead-simple chess eval might look like:

```python
def evaluate(board):
    piece_values = {'P': 1, 'N': 3, 'B': 3, 'R': 5, 'Q': 9, 'K': 0}
    score = 0
    for piece in board.white_pieces:
        score += piece_values[piece.type]
    for piece in board.black_pieces:
        score -= piece_values[piece.type]
    return score
```

Positive score = white is winning. Negative = black is winning. Zero = roughly even.

Real eval functions get more sophisticated (piece positioning, king safety, pawn structure, etc.), but the core idea is the same: **compress a complex state down to a single number**.

**Analogy:** Think of it like a health check script for a server. You can't know everything about the system's state at a glance, so you sample CPU load, memory usage, disk I/O, and combine them into a single "health score" from 0-100. That scoring function *is* an evaluation function. It's a heuristic -- not perfect, but good enough to make decisions with.

The concept shows up everywhere:

- **Game AI (minimax):** Score board positions so the engine picks the best move.
- **Search/optimization:** Score candidate solutions so you know which direction to explore.
- **Machine learning:** Loss functions are essentially eval functions in reverse -- they score how *bad* a model's predictions are, and training minimizes that score.
- **Static analysis:** Linters "evaluate" code quality by scoring patterns against rules.

The quality of your eval function basically determines how smart your system behaves. A chess engine with a naive "just count pieces" eval will play worse than one that also considers board control, even if the search algorithm is identical. Garbage in, garbage out.
