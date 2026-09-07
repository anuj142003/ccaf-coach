# Mechanism inventory — READ BEFORE AUTHORING ANY ENFORCEMENT OR CONFIGURATION ITEM

**Why this file exists.** A coach authoring their own items can only test whether the
candidate agrees with the coach. If a real mechanism is missing from the coach's
option lists, no number of items will ever detect the candidate not knowing it —
and the coach will confidently report the cluster as mastered. This happened, and
was caught only when a real score report contradicted the coach.

**The rule: every option in an enforcement or configuration item must name a
mechanism from the lists below, using the name the documentation uses.** Never
invent a paraphrase like "a prerequisite gate in code" — that phrasing is not a
Claude Code mechanism, and a candidate who learns it will not recognise the real
answer on the exam.

Verified against official docs, September 2026:
- Hooks reference — https://code.claude.com/docs/en/hooks
- Configure permissions — https://code.claude.com/docs/en/permissions
- Agent SDK permissions — https://code.claude.com/docs/en/agent-sdk/permissions

Re-verify before a teaching session if the docs may have moved on.

---

## 1. Claude Code guidance and enforcement ladder

Ordered from "model decides" to "system decides". **The exam's central
discrimination is where a given requirement sits on this ladder.**

| Mechanism | Loads / fires when | Enforcement strength |
|---|---|---|
| `CLAUDE.md` | Always, at session start | Model discretion |
| `@import` in CLAUDE.md | Always (modularity only, not conditionality) | Model discretion |
| `.claude/rules/*.md` with `paths:` glob | When a matching file is touched | Model discretion |
| Skills (`.claude/skills/`) | On demand, by name or description match | Model discretion |
| Slash commands (`.claude/commands/`) | When the user invokes them | Model discretion |
| `PreToolUse` hook | Before a tool call executes | **Deterministic — can block** |
| `PostToolUse` hook | After a tool call succeeds | **Deterministic — cannot undo** |
| Settings permissions (`allow` / `ask` / `deny`) | On every tool call | **Deterministic — hard allow/deny** |

**Model discretion means it can be skipped.** Any stem describing an instruction
that "is followed inconsistently", "is skipped occasionally", or "holds most of the
time" is telling you the requirement is on the wrong rung. The fix moves it down
the ladder — it is never "restate it more emphatically in the prompt", and it is
never "add few-shot examples".

The docs state the converse too: for instructions that never change, prefer
CLAUDE.md — it loads without running a script and is the standard place for static
project conventions. Not everything belongs in a hook.

---

## 2. Hooks — what each event can actually do

Full event list is in the hooks reference; these are the exam-relevant ones.

- **`PreToolUse`** — fires *before* a tool call executes and **can block it**.
  Returns `hookSpecificOutput.permissionDecision` of `allow` / `deny` / `ask` /
  `defer`, plus `permissionDecisionReason`. Exit code 2 also blocks. This is the
  mechanism for "X must happen / must be true before the action, every time".
- **`PostToolUse`** — fires *after* a tool call **succeeds**. **Cannot block or
  undo** the call; exit 2 only surfaces stderr to Claude. This is the mechanism
  for "X must happen after the action, every time" — formatting, linting, test
  runs, audit or notification writes.
- **`PostToolUseFailure`** — fires after a tool call *fails*. Note `PostToolUse`
  does **not** fire on failure; a stem about handling failed calls needs this event.
- **`PermissionRequest`** — fires when a call needs a permission decision.
- **`Stop` / `SubagentStop`** — exit 2 prevents stopping and continues the turn.
  Relevant to "every session must end in resolution or escalation".
- **`SessionStart`** — fires on start *and on resume* (matcher values include
  `startup`, `resume`, `clear`, `compact`, `fork`), so it can re-inject state.
- **`PreCompact` / `PostCompact`** — around context compaction.

**Configuration shape:** hook event → matcher group → hook handler. Matchers filter
on tool name for tool events (`Bash`, `Edit|Write`, `mcp__memory__.*`). The
optional `if` field uses permission-rule syntax (`Edit(*.ts)`, `Bash(git *)`).
Handler types: `command`, `http`, `mcp_tool`, `prompt`, `agent`.

**Where hooks live, and whether they are shareable:**

| Location | Scope | Shareable |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No |
| `.claude/settings.json` | One project | **Yes — commit it** |
| `.claude/settings.local.json` | One project | No, gitignored |
| Managed policy settings | Organization-wide | Yes, admin-controlled |
| Plugin `hooks/hooks.json` | While plugin enabled | Yes |
| Skill / subagent frontmatter | While invoked / running | Yes |

Hooks also fire inside subagents, carrying `agent_id` and `agent_type`.

**Two traps worth building items around:**
- The `if` filter is best-effort. The docs say to use the **permission system**,
  not a hook, to enforce a hard allow or deny.
- A timed-out `PreToolUse` command hook **does not** block the call. The docs say
  not to rely on a stalled hook as a gate.

---

## 3. Settings permissions

Live under the `permissions` key of a settings.json, at the same four scopes as
hooks (user / project / local / managed).

Three rule types, in `Tool(specifier)` form:
- **`allow`** — use the tool without manual approval
- **`ask`** — prompt for confirmation
- **`deny`** — prevent use entirely

**Evaluated deny → ask → allow.** A deny beats any matching allow, and holds even
in `bypassPermissions`.

**Permission modes:** `default`, `plan`, `acceptEdits`, `auto`, `dontAsk`,
`bypassPermissions`.

**Agent SDK evaluation order** (useful for stems set in the SDK rather than the
CLI): hooks first → deny rules → permission mode → allow rules → `canUseTool`
callback. A hook returning `allow` does **not** skip the deny and ask rules.

**Use permissions, not a hook, when the requirement is a hard prohibition** — "this
tool must never be used on these paths", "this command must never run". Use a
`PreToolUse` hook when the requirement is a *conditional check* that needs custom
logic before the call proceeds.

---

## 4. API-plane mechanisms — do not mix planes carelessly

`tool_choice`, the Batches API and JSON-schema tool use are **Messages API**
parameters. Hooks and settings permissions are **Claude Code / Agent SDK**
mechanisms. A stem set in one plane should generally offer options from that plane.

Mixing them in one option list is legitimate only when the scenario plainly spans
both — and when it does, be explicit in the grading which plane the answer sits in.

**`tool_choice` facts:**
- `"auto"` — the model may return prose instead of calling a tool
- `"any"` — the model must call *some* tool
- `{"type":"tool","name":...}` — the model must call *that* tool
- **It constrains one turn.** A turn is one model response; a tool call spans two
  or more turns. `tool_choice` cannot hold an invariant across a loop.

**Sequencing prerequisite data before dependent tools** (a named objective) has
answers on both planes. On the API plane: force the prerequisite tool on the
opening turn, then let the dependent call happen on a following turn. Where the
requirement must hold across many iterations, the answer is a **`PreToolUse`
hook** or a **deny rule**, not `tool_choice`.

---

## 5. Coverage self-audit — run this before declaring any cluster cleared

For each mechanism in §1–§4, ask:

1. Has it appeared as a **correct answer** in at least one item?
2. Has it appeared as a **distractor** where a neighbouring mechanism was correct?

A mechanism that has never been the correct answer is **untested**, however many
items the candidate has answered in that area. Log it as untested, not as cleared.

The specific pairs that must each have been the correct answer at least once
before the configuration-and-enforcement cluster may be called cleared:

- `PreToolUse` (before, conditional logic) vs `PostToolUse` (after)
- `PostToolUse` vs `PostToolUseFailure` (the call failed)
- Hook (conditional check) vs `deny` rule (hard prohibition)
- Hook or permission (must hold) vs CLAUDE.md (static convention, discretion fine)
- `.claude/rules/` glob (conditional load) vs `@import` (modularity, still always loads)
- Skill vs slash command vs subagent
- Project `.claude/settings.json` vs `~/.claude/settings.json` vs `settings.local.json`
- `tool_choice` (one turn) vs hook (every turn)
