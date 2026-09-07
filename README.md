# CCA-F Coach

A Claude Code / Claude Desktop **skill** that runs an adaptive study-and-test
program for the **Claude Certified Architect – Foundations** exam (code CCAR-F).

Claude acts as examiner and coach: it generates every question live against the
exam blueprint, grades your reasoning rather than your letter, tracks per-objective
coverage across sessions, and tells you when the readiness gates are met — and
when they are not.

There is **no question bank**. That is deliberate; see [Why no question
bank](#why-no-question-bank).

## Install

```bash
git clone https://github.com/anuj142003/ccaf-coach.git ~/.claude/skills/ccaf-coach
```

Then start a session:

```
Use the ccaf-coach skill. I sat CCA-F and failed — here's my score report.
```

Any phrasing about preparing, drilling, mock-examining or checking readiness for
CCA-F / CCAR-F triggers it. Have your official exam guide handy: if you supply it,
it overrides the skill's built-in blueprint.

## How a session runs

Every session is five phases, ~8 graded items:

1. **Cold open** — re-test whatever the log owes a cold re-test, on unfamiliar
   scenarios, no teaching first.
2. **New ground** — the highest-priority untested objective or cluster.
3. **Contrast probes** — after a correct answer, one detail flips so the correct
   mechanism changes. Correct-but-unprobed is not cleared.
4. **Teach only confirmed gaps**, then re-test on a fresh scenario.
5. **Close** — cold score, reasoning score, dominant failure pattern, gate status,
   and a resume block you paste into the next session.

For each item you commit **a letter, why it beats the closest rival, which detail
eliminates which option, and your confidence**. Nothing is revealed until you
commit. One candidate's reasoning score went from 3/8 to 7/8 once the elimination
mapping was enforced.

## Starting points

| You are | Path |
|---|---|
| A retaker with a score report | Transcribe per-objective scores, target *clusters* of weak objectives, open cold |
| Sat it before, or know the material | 15-item cold diagnostic across all domains, then as above |
| New to the material | Syllabus in domain-weight order, teach-then-test, diagnostic once covered |

Self-assessment is treated as a hypothesis, not data — one candidate who reported
being "new to the material" scored 93% cold.

## Readiness gates

Claude will not call you ready until all five hold:

1. Two consecutive sessions at **≥80% cold** first-encounter accuracy.
2. Reasoning sound on **≥75%** of cold items.
3. **No cluster below 75%**, every confirmed gap cleared on a cold re-test.
4. The `mechanism-inventory.md` **coverage audit passes** — every mechanism
   relevant to your weak objectives has been the correct answer at least once.
5. One **20-item timed set (40 min) at ≥80%**, attempted only once 1–4 are met.

Gates are reported at every close and are never waived for being close. They
inform your decision; they don't own it.

## Why no question bank

The skill has failed twice, and both remedies are load-bearing rules now:

- **The answer leaked through form.** In a static bank, the correct option was the
  longest 92% of the time (208 chars vs 106). A candidate scored 92% without
  reading the questions. Hence the length-parity and letter-rotation checks in §7.
- **The answer was never on the page.** Across ~115 self-authored items on
  multi-step enforcement, the coach never once offered `PreToolUse` or settings
  permissions as an option. The candidate scored 9/9, was told the cluster was
  closed, and scored **0% on the four exam objectives covering that ground.**
  Hence `references/mechanism-inventory.md`, which must be read before authoring
  any enforcement or configuration item.

## Repository layout

| File | Purpose |
|---|---|
| `SKILL.md` | The program itself — protocol, question-construction rules, failure taxonomy, scoring gates |
| `references/objectives.md` | 29 objective titles transcribed verbatim from a real score report; the coverage-tracking list |
| `references/mechanism-inventory.md` | Doc-verified mechanism names. Read before authoring enforcement or configuration items |
| `references/content-reference.md` | Domain facts, in/out-of-scope lists — fallback content |

## Sources of truth, in order

1. Your own exam guide, if supplied — always wins.
2. `references/mechanism-inventory.md`.
3. §11 derived teaching insights.
4. `references/objectives.md`.
5. `SKILL.md` §4 and `references/content-reference.md` — fallback only, from guide
   v1.0 (July 2026) with corrections applied September 2026.

Blueprint weights in §4 rest on a v1.0 transcription and are unverified against any
score report — supply your guide if you have one.
