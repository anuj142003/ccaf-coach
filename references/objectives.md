# Test objectives — transcribed from a real CCAR-F score report

**Source:** score report for a passing attempt, exam code CCAR-F, dated
5 September 2026. These are the objective titles the score report itself prints,
**verbatim**. They are the most authoritative objective list this skill has,
because they come from the scoring system rather than from a transcription of a
guide.

**Use these for coverage tracking, in preference to the task-statement shorthand
in SKILL.md §4.** A candidate's own guide still wins over both.

**What this list does not tell you:** the report prints no domain grouping, no
weights, and no per-objective item counts. The domain names and percentage weights
in SKILL.md §4 come from guide v1.0 and are **unverified against this report**.
Treat the groupings below as inferred, and say so to candidates.

**Observed count: 29 objectives** across roughly 60 items — so most objectives
carry one or two items. Apply the noise caveat in SKILL.md §3.

---

## Agentic architecture and orchestration

1. Design orchestration-layer safeguards ensuring every agent session ends with a completed resolution or human escalation, regardless of how the agentic loop terminates.
2. Design structured handoff packages that preserve accumulated context, findings, and authorization state when transferring control between agent steps or to a human operator.
3. Apply session resumption techniques — including targeted re-analysis of changed files and context injection — to restore agent state accurately without repeating prior work.
4. Decompose complex tasks into dynamically generated subtasks that adapt as new information is discovered, rather than executing a fixed sequence regardless of intermediate findings.
5. Apply PreToolUse and PostToolUse hook patterns to enforce business rules and policy constraints not delegated to model discretion.
6. Explain how the agentic loop uses model responses and stop_reason signals to decide whether to continue tool execution or terminate and return a final response.

## Claude Code configuration and workflows

7. Create and deploy custom slash commands in the correct project or user directory so they are available to intended users and invoked on demand for task-specific workflows.
8. Structure iterative refinement workflows by providing concrete input-output examples, targeted feedback on specific failures, and batched issue descriptions for consolidated evaluation.
9. Determine when to use plan mode versus direct execution based on task scope, reversibility, architectural uncertainty, and the need for stakeholder review before implementation.
10. Distinguish instructions that must be enforced through settings permissions or hooks from those appropriately placed in CLAUDE.md, and restructure configurations accordingly.
11. Select the correct Claude Code configuration mechanism — CLAUDE.md, .claude/rules/ with glob patterns, Skills, hooks, or settings permissions — based on guidance type and when it should apply.
12. Implement PostToolUse hooks that automatically enforce code quality constraints — such as formatting, linting, or test execution — after every file edit, independent of model instruction-following.
13. Apply systematic codebase exploration strategies using Grep, Glob, and Read tools that build incremental understanding while managing context window constraints.
14. Configure MCP servers and project settings at the correct scope — project-level for shared team tooling and user-level for personal or experimental configurations.
15. Apply context management strategies — subagent isolation, scratchpad files, and targeted file reading — to sustain coherent codebase exploration across sessions exceeding context limits.
16. Select the appropriate method for providing project context to Claude Code — @ references, CLAUDE.md, or inline description — based on reusability, specificity, and cross-session need.

## Context, state and escalation

17. Explain why conversation history must be explicitly included in each API request and identify the correct mechanism for maintaining state across multiple turns in a stateless API.
18. Apply escalation decision criteria to determine when an agent should immediately honor a human escalation request versus attempt autonomous resolution using available tools.
19. Design human review routing strategies that direct extractions to reviewers based on confidence scores, document characteristics, and field-level ambiguity rather than random sampling.

## Prompt engineering and structured output

20. Select the appropriate API processing mode — synchronous Messages API or asynchronous Message Batches API — based on latency requirements, workflow blocking behavior, and acceptable processing windows.
21. Select and implement the most reliable structured output method — tool use with JSON schema, prompt-based formatting, or prefilled responses — based on required schema compliance strictness.
22. Design extraction schemas with optional fields, nullable values, and appropriate enum definitions that allow the model to accurately represent missing or ambiguous information without fabricating values.
23. Implement tool use with defined JSON schemas to enforce structured output compliance, and configure tool_choice to guarantee tool invocation when conversational responses would cause downstream failures.
24. Apply extraction accuracy patterns — structured schemas with optional fields, format normalization instructions, and few-shot examples — to reduce hallucination and improve consistency across varied document formats.
25. Design feedback loop mechanisms that capture structured metadata about model errors and use those patterns to improve prompts, schemas, or few-shot examples in future iterations.

## Tool design and MCP integration

26. Implement MCP tool error handling that surfaces structured, type-specific error information to the agent — including recoverability status and suggested next actions — rather than generic failure messages.
27. Improve tool selection reliability by expanding tool descriptions with use-case examples, input format specifications, and explicit disambiguation guidance for semantically similar tools.
28. Integrate MCP servers into Claude Code and agent applications by selecting the correct server scope, configuring authentication via environment variable expansion, and verifying tool discovery.
29. Configure the tool_choice parameter to guarantee tool invocation when structured output is required, and sequence multi-tool workflows so prerequisite data is obtained before dependent tools are called.

---

## Objectives that name a mechanism explicitly

Nine objectives name specific mechanisms in their titles: 5, 10, 11, 12, 13, 14,
16, 23, 29. **For each of these, every named mechanism must appear as the correct
answer to at least one item before the objective may be called covered.** Read
`mechanism-inventory.md` before authoring any of them.

Objectives 5, 10, 11 and 12 together form the configuration-and-enforcement
cluster. In the calibration case a candidate scored 0% on 5, 10, 11 and 29 while the coach's own items had reported the same ground as
mastered — because the coach's option lists never once named `PreToolUse` or
settings permissions.
