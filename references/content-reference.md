# Content reference — FALLBACK. The candidate's own guide always wins.

Facts transcribed from exam guide v1.0 (July 2026), with the Claude Code
configuration section corrected against primary docs in September 2026. Where a
candidate's guide differs, theirs wins. The teaching insights in SKILL.md §11 stay
regardless of guide version.

**Before authoring any enforcement or configuration item, read
`mechanism-inventory.md`.** This file states facts; that file states which
mechanism names must appear in option lists.

---

## D1 — Agentic architecture and orchestration

Continue while `stop_reason == "tool_use"`, stop on `"end_turn"`. Text and
`tool_use` can co-occur in one response, so non-empty text is **not** a stop
signal. Append tool results to history — failing to do so produces **repetition,
not early termination**. Anti-patterns: iteration caps, parsing response text for
completion phrases.

Hub-and-spoke coordinator routes all communication and errors, decomposes,
delegates, aggregates, and selects subagents by complexity. **Subagents have
isolated context and inherit nothing** — no shared memory, no session to link.
Incomplete coverage usually means the coordinator's decomposition was too narrow;
synthesis can only combine what was found. Spawn via `Task`; the coordinator's
`allowedTools` must include `"Task"`; parallelise with multiple `Task` calls in
**one** response. Pass complete prior findings plus source metadata explicitly.
Delegate **goals and quality criteria, not procedures**, preserving control through
a required output contract, scope boundaries, and routing through the coordinator.
Decomposition should be **dynamic** — subtasks generated as findings arrive, not a
fixed sequence executed regardless of what is discovered.

Sessions: `--resume`, `fork_session` to branch divergent approaches from a shared
baseline, the **manifest pattern** for crash recovery (agents export state; the
coordinator loads a manifest on resume and injects it), checkpoint completed work.
Prefer a fresh session with a structured summary over resuming on stale tool
results, and tell a resumed session which files changed. `SessionStart` fires on
resume as well as startup, so it can re-inject state.

Every session must end in a completed resolution or a human escalation, whatever
happens to the loop — `Stop` and `SubagentStop` hooks can prevent a premature stop.

Handoffs — to another agent step or to a human — must carry accumulated context,
findings **and authorization state**, not just a summary.

Enforcement: hooks and permissions are deterministic, prompts are probabilistic.
See `mechanism-inventory.md` for which mechanism fits which requirement.

## D2 — Tool design and MCP integration

Tool **descriptions** are the primary selection signal: inputs, examples, edge
cases, boundaries, when to use versus similar tools. Fix misrouting by rewriting,
renaming or splitting descriptions — **not** with a routing classifier, and not
with few-shot first. A keyword-sensitive system prompt can override good
descriptions; if the stem says the description is already thorough, the prompt is
the defect.

Errors: `isError` plus `errorCategory` (transient / validation / business /
permission) plus `isRetryable` plus a readable message. Generic "operation failed"
blocks recovery. Distinguish an access failure from a valid empty result — **and a
record that exists but is archived or otherwise unusual is a valid result carrying
its status, not an error.** Recover transient errors locally and propagate only
the unresolvable, with partial results and what was attempted.

Fewer tools select better (4–5, not 18); none outside an agent's role; scope
cross-role tools narrowly for high-frequency needs.

`tool_choice`: `"auto"` may return prose · `"any"` must call some tool · forced
`{"type":"tool","name":...}` must call that one. **It constrains one turn.**

MCP: project `.mcp.json` (shared, version-controlled) vs user `~/.claude.json`
(personal); `${TOKEN}` expansion keeps secrets out of the repo; all servers' tools
are discovered at connection time; **resources** expose content catalogs to cut
exploratory calls while **tools** act; rich MCP descriptions stop the agent
preferring built-in Grep; prefer community servers for standard integrations.

## D3 — Claude Code configuration and workflows

**Read `mechanism-inventory.md` in full before authoring here. This is the section
that was wrong.**

The organising question is *where on the guidance-to-enforcement ladder does this
requirement belong?* — not *which file is nicest*.

- **`CLAUDE.md`** — always-loaded static conventions, model discretion. Hierarchy:
  user `~/.claude/CLAUDE.md` (personal, **not** shared through version control) /
  project `.claude/CLAUDE.md` / directory-level. `@import` gives modularity but
  everything still loads every session. `/memory` shows what is loaded — a
  teammate missing instructions has them in user scope. `@` references, CLAUDE.md
  and inline description are chosen on reusability, specificity and cross-session
  need.
- **`.claude/rules/`** with YAML `paths:` globs — conditional loading when a
  matching file is touched. Better than a directory-level CLAUDE.md for a file
  type spread across a repo, and better than `@import` when the point is *not
  loading it the rest of the time*.
- **Skills** (`.claude/skills/`, frontmatter `context: fork`, `allowed-tools`,
  `argument-hint`, and hooks) — on-demand workflows needing isolation or tool
  restriction. `context: fork` isolates output in a subagent, so it is **wrong
  when the result must land in the main conversation**.
- **Slash commands** (`.claude/commands/` shared vs `~/.claude/commands/`
  personal) — repeatable prompts invoked by name.
- **`PreToolUse` hooks** — deterministic, fire before a call, **can block it**.
  The mechanism for a business rule or policy constraint that must hold before an
  action, every time, not delegated to model discretion.
- **`PostToolUse` hooks** — deterministic, fire after a call succeeds, **cannot
  undo it**. The mechanism for formatting, linting, test execution or audit
  writes after every edit, independent of instruction-following. Note
  `PostToolUseFailure` for failed calls.
- **Settings permissions** (`allow` / `ask` / `deny`, evaluated deny → ask →
  allow) — hard allow and deny of tool use. The mechanism for an absolute
  prohibition. The docs specifically say to use permissions rather than a hook to
  enforce a hard allow or deny.

Anything described as "in the prompt and sometimes skipped" is on the wrong rung.
Anything static and universal belongs in CLAUDE.md and does **not** need a hook.

Plan mode vs direct execution vs multi-phase: see SKILL.md §11 — decided on
scope, reversibility, architectural uncertainty, and whether stakeholder review is
needed before implementation.

Iterative refinement: 2–3 concrete input/output examples beat prose; targeted
feedback on the specific failure beats general instruction; **batch issue
descriptions for consolidated evaluation** — combine interacting fixes in one
message, send independent fixes separately, interacting first.

CI: `-p` / `--print` for non-interactive runs (prevents hangs); permission modes
plus **cost and turn limits** stop runaways; `--output-format json` with
`--json-schema` for parseable findings; an independent review instance beats
self-review; pass prior findings so it reports only new issues; give test
generation the existing tests, fixture conventions, and criteria separating
behavioural tests from trivial assertions.

Exploration: Grep, Glob and Read build incremental understanding. See the tool
composition section of SKILL.md §11.

## D4 — Prompt engineering and structured output

Explicit categorical criteria beat "be conservative" or "only high-confidence". A
high-false-positive category erodes trust in the accurate ones — disable and
rebuild with explicit criteria. Supply project conventions, accepted patterns and
exclusion criteria as persistent context. To learn *which* patterns produce false
positives, capture structured metadata — a `detected_pattern` field — and analyse
it across a sample; that is diagnosis, distinct from the remedy.

Few-shot is the strongest lever when detailed instructions still produce
inconsistency: 2–4 targeted examples across **varied document structures**,
enabling generalisation to novel patterns; include one where a field is genuinely
absent so the model returns null; **pair with explicit format-normalisation rules**
when source formats are heterogeneous. Inconsistent-but-schema-valid values are
fixed at extraction this way, not by post-processing and not by retry — nothing is
invalid, so there is nothing to retry against.

`tool_use` with a JSON schema is the most reliable structured-output method,
above prompt-based formatting and prefilled responses. It guarantees schema-valid
output and eliminates **syntax** errors but **not semantic** ones (line items not
summing, values in the wrong field).

Make fields **optional or nullable** when the source may lack them; required
fields force fabrication. Enums need `"other"` with a detail field for real values
outside the list, and `"unclear"` for genuine ambiguity. Nullable is for absence;
`"other"` is for an unlisted real value — different defects.

Retry with the specific validation error appended fixes format and structural
errors but **not information absent from the source**.

Batches API: 50% cheaper, up to 24h, **no latency SLA**, no tool calling inside a
job, `custom_id` correlates results **and identifies failures for resubmission**.
Right for overnight and periodic work; wrong for anything blocking a person, and
wrong for anything needing a tool call mid-task. Those are the only two
disqualifiers.

Multi-pass review: an **independent instance** beats self-review, "be critical"
instructions, or extended thinking; split large reviews into per-file passes plus a
cross-file integration pass; a larger context window does **not** fix attention
dilution; truncation → split into scoped calls and merge, don't raise `max_tokens`;
consensus-of-two-runs suppresses intermittently-caught bugs.

## D5 — Context management and reliability

Progressive summarization loses amounts, dates and IDs — keep a **persistent
case-facts block outside the summarized history**; summarizing *more* frequently
makes it worse. Counter lost-in-the-middle by putting key findings **first** with
section headers; trim verbose tool outputs. Conversation history must be included
explicitly in each API request — the API is stateless.

Escalation — closed set of triggers: **explicit human request · policy gap or
ambiguity that leaves the correct action undetermined · inability to make
progress.** Not complexity, not sentiment, not self-reported confidence. Honour an
explicit request for a human immediately. The union matters: one trigger is
enough, and the absence of the others disqualifies nothing. **Ambiguity you can
resolve by asking is not an escalation** — multiple matches → request another
identifier. Asking is progress; escalate when nothing is left to ask or try.
Never close a conversation as resolved when nothing was resolved.

Error propagation: structured context (failure type, attempted query, partial
results, alternatives). Anti-patterns: generic statuses, empty-as-success,
terminating a whole workflow on one failure. Annotate coverage gaps.

Large codebases: degradation shows as citing "typical patterns" instead of
specifics found earlier — scratchpad files, subagent isolation for verbose
exploration, phase summaries before spawning the next phase, `/compact`.

Human review: emit field-level confidence and route on confidence, document
characteristics and field-level ambiguity **rather than random sampling**;
calibrate thresholds on a **labelled validation set**. Separately, stratified
sampling of *auto-accepted* output is how you audit whether the accepted band is
as good as believed — routing allocates scarce capacity, sampling measures.
Complements, not alternatives.

Provenance: preserve claim-to-source mappings (document, excerpt, date) through
synthesis, since attribution is lost when summarization compresses findings;
require publication and collection dates; separate well-established from contested
findings with methodological context. Render financial data as tables, news as
prose, technical findings as lists.

---

## Out of scope — never test

Fine-tuning · auth and billing · MCP hosting and infrastructure · model internals ·
Constitutional AI and RLHF · embeddings and vector databases · computer use ·
vision · streaming · rate limits and pricing · prompt-caching internals ·
tokenization · cloud-provider specifics.
