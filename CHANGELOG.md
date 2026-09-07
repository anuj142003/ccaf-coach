# ccaf-coach — v2 changes and the evidence behind them

## The calibration case

One candidate ran four coaching sessions with v1 of this skill and then sat the
exam. **Result: 865 against a 720 cut — a comfortable pass.** The per-objective
breakdown is what makes the case useful, because it is the first time this
program's judgements could be checked against an external scorer.

**Where the coach said "cleared cold", the exam said 0%.**

| Objective (verbatim from the score report) | Exam | Coach's verdict |
|---|---|---|
| Apply PreToolUse and PostToolUse hook patterns to enforce business rules and policy constraints not delegated to model discretion | **0%** | "sequencing enforcement 9/9, closed cold" |
| Distinguish instructions enforced through settings permissions or hooks from those appropriately placed in CLAUDE.md | **0%** | not separately tracked |
| Select the correct Claude Code configuration mechanism — CLAUDE.md, .claude/rules/ globs, Skills, hooks, settings permissions | **0%** | "rules-glob, context: fork, commands scope — cleared" |
| Configure tool_choice and sequence multi-tool workflows so prerequisite data is obtained before dependent tools are called | **0%** | "closed cold" |

Meanwhile the clusters the coach's final cheatsheet named as the candidate's live
risk — escalation, MCP error handling, format normalisation, nullable vs enum,
Batches — all came back at **100%**. The diagnosis was close to inverted precisely
where it was most confident.

Three further objectives came back at 50%: adaptive decomposition, Grep/Glob/Read
exploration, and context management. Milder versions of the same thing.

## Root cause

**The coach never once named `PreToolUse` or settings permissions in an option
list.** Across roughly 115 authored items, every enforcement question offered
`tool_choice`, a `PostToolUse` hook, "a prerequisite gate in code", or a prompt
restatement. "Prerequisite gate in code" is the coach's own phrasing, not a
Claude Code mechanism.

So the coach taught a decision procedure — *before-and-in-a-loop → gate in code* —
that routed the candidate **away** from the documented answer, then graded him
correct nine consecutive times for following it. The candidate's 9/9 and the
exam's 0% are not in conflict: they measured different things.

**The generalisable failure: a self-authored item set can only measure whether the
candidate agrees with the author.** Where the author's model of the domain is
wrong, the items are structurally blind to it, and a confident "cleared cold" is
worse than no measurement at all, because it tells the candidate to stop looking.

v1 §1 already warned that LLM-authored multiple choice leaks answers through
elaboration. That was one failure mode of self-authored items. This is the second,
and it is the more dangerous one because it is invisible from inside the program.

## Changes in v2

**New `references/mechanism-inventory.md`** — the canonical, doc-verified list of
Claude Code guidance and enforcement mechanisms, with the rule that every option in
an enforcement or configuration item must name a mechanism from it, using the
documentation's own name. Includes a coverage self-audit and the eight contrast
pairs that must each have been a correct answer before that cluster may be called
cleared. Verified September 2026 against the hooks reference, the permissions page
and the Agent SDK permissions page; each is cited in the file.

**New `references/objectives.md`** — all 29 objective titles transcribed verbatim
from the score report. More authoritative than the task-statement shorthand in
§4, because it comes from the scoring system rather than a guide transcription.
Flags that the report prints no domains, no weights and no item counts, so v1's
domain weights are unverified.

**New `references/content-reference.md`** — the former §12, with the Claude Code
configuration section rewritten around the enforcement ladder, and smaller
corrections to error handling (a valid-but-unusual record is not an error),
normalisation (fixed at extraction, not post-processing), enum vs nullable, and
escalation (ambiguity you can resolve by asking is not an escalation).

**SKILL.md §1** — second lesson added: self-authored blind spots, with the
inventory rule as the remedy.

**SKILL.md §7** — new construction rule requiring option lists to be drawn from
the mechanism inventory, plus a check for whether the item's plane (Claude Code
vs Messages API) is consistent.

**SKILL.md §9** — gate language reframed. A cluster tested only against
self-authored items is now reported as *held under my items*, never *cleared*.
Gate 3 now additionally requires the mechanism-inventory coverage audit to pass.
Added the honesty requirement to state the ceiling of self-authored measurement
when reporting readiness.

**SKILL.md §5** — hard stop rule. In the calibration case the coach advised
stopping twice, the candidate continued, and the sixth block of a 60-item set —
run near 1 a.m. — scored 5/10 against 7–9 for the preceding blocks. Four of the
five misses were pairs answered correctly earlier the same evening. All six
re-tested correct when rested. Fatigue is now recorded separately by default, as
targeted sets already were.

**SKILL.md §10** — elapsed-time rule. The coach twice misjudged how much real time
had passed, once nearly discarding a valid six-day-separated cold re-test as
"recall" and once assuming a multi-day thread was a single sitting. Check the date
before classifying a re-test.

**SKILL.md §13** — the cheatsheet section now warns that a refresher built from
the coach's own logged misses inherits the coach's blind spots, and must include
the objectives never tested rather than only the ones missed.

## What v2 does not fix

The core limitation is structural and cannot be removed by editing this file: a
coach authoring its own items cannot detect a gap in its own model of the domain.
The inventory narrows it for configuration and enforcement, which is where the
observed damage was, but the same failure could sit in any objective whose correct
mechanism is absent from the coach's option lists.

**Mitigations worth adopting at org level:**
- Have the coach read primary docs before authoring in any mechanism-naming
  objective (5, 10, 11, 12, 13, 14, 16, 23, 29), not from recall.
- Collect per-objective score reports from everyone who sits, and compare them
  against what the log claimed. Two or three more data points would show whether
  the config-and-enforcement blindness was the only one.
- Treat this skill's readiness gates as necessary, never sufficient.
