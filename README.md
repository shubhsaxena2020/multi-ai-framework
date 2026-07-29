# multi-ai-framework

A Claude Code skill that turns Claude Code into a **coordinator** for a small team of independently-billed AI CLIs (in the reference setup: Hermes Agent, OpenAI Codex CLI, OpenCode, Google Antigravity) instead of doing all the heavy research/review/build work itself.

The core idea: Claude Code's own usage is billed to your plan; other CLIs you already pay for or run for free are not. Route deep research, adversarial review, and large from-scratch generation to them — but never trust a delegate's claim as fact until it's independently checked against real evidence. A subagent's output is a hypothesis, not a result.

## Why this exists

Two problems, both learned the hard way across real sessions (see `EVOLUTION_PLAN.md` and `TEST_SUITE.md`'s incident table for the specifics):

1. A single coordinator doing all its own diagnostic legwork (log-diving, root-causing bugs solo) wastes the coordinator's own budget on work a delegate could do just as well.
2. A delegate's confident-sounding claim is not automatically true — and two delegates *agreeing* isn't either, because different models share correlated blind spots. Verification against a primary source is mandatory, not optional.

## What's in here

- [`SKILL.md`](SKILL.md) — the skill's entry point (frontmatter + quick start), what Claude Code loads first
- [`SKILL_BRIEF.md`](SKILL_BRIEF.md) — the one-paragraph mental model and why this exists
- [`REQUIREMENTS.md`](REQUIREMENTS.md) — what invoking this framework actually obligates the coordinator to do (R1–R9)
- [`BEHAVIOR_SPEC.md`](BEHAVIOR_SPEC.md) — the "boss, not worker" posture in concrete behavioral terms
- [`WORKFLOW.md`](WORKFLOW.md) — the step-by-step sequence for a delegated task
- [`TOOLS_AND_MCP.md`](TOOLS_AND_MCP.md) — per-CLI invocation notes and known quirks (Windows-path examples from the reference setup — adapt paths/binaries to your own machine and installed tools)
- [`PROMPT_ARCHITECTURE.md`](PROMPT_ARCHITECTURE.md) — how to write a delegation prompt that gets real, checkable work back
- [`CONTEXT_MODEL.md`](CONTEXT_MODEL.md) — what state carries across a delegated call, what doesn't
- [`QUALITY_GATES.md`](QUALITY_GATES.md) — the checks a delegated result must pass before you act on it
- [`FAILURE_AND_RECOVERY.md`](FAILURE_AND_RECOVERY.md) — known failure modes (hangs, stale claims, hallucination) and the fix for each
- [`TEST_SUITE.md`](TEST_SUITE.md) — how this skill was validated against real incident history, and how to re-validate it
- [`EVOLUTION_PLAN.md`](EVOLUTION_PLAN.md) — what's deliberately out of scope, the trigger to revisit each decision, and the research/rebalancing history behind this skill's current shape

## Using it

1. Copy this directory into your Claude Code skills folder (typically `~/.claude/skills/multi-ai-framework/` on the machine or account where Claude Code runs).
2. Invoke it by saying "use multi AI framework" (or similar — "deep work", "heavy work") in a Claude Code session, or let Claude Code pick it up automatically per its `SKILL.md` description.
3. Edit `TOOLS_AND_MCP.md` to match the actual CLIs you have installed — the reference version documents Hermes Agent, OpenAI Codex CLI, OpenCode, and Google Antigravity's `agy` CLI on Windows, but the framework's core contract (diagnosis delegates, mutation stays with the coordinator, every claim gets verified) applies to any set of delegate tools.

## Adapting this to your own setup

This skill was built and refined against one specific machine's real incident history — the tool names, binary paths, and specific bugs in `TOOLS_AND_MCP.md` and `FAILURE_AND_RECOVERY.md` reflect that setup, not universal facts. Treat `REQUIREMENTS.md`, `BEHAVIOR_SPEC.md`, `WORKFLOW.md`, and `QUALITY_GATES.md` as the portable core; treat `TOOLS_AND_MCP.md` as a template to replace with your own tools' real invocation details.

## License

MIT — see [`LICENSE`](LICENSE).
