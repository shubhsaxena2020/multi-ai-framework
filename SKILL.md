---
name: multi-ai-framework
description: Use when the user says "use multi AI framework" or asks for deep/heavy work across research, review, or building — orchestrates Claude Code as coordinator delegating to Hermes, Codex, OpenCode, and Antigravity, with mandatory independent verification of every claim before acting on it.
---

# Multi-AI framework

Full specification lives in this skill's companion documents — read the one relevant to the question at hand rather than re-deriving it:

- `SKILL_BRIEF.md` — what this is, why it exists, one-paragraph mental model
- `REQUIREMENTS.md` — what "using the framework" actually obligates you to do
- `BEHAVIOR_SPEC.md` — the hardcore-boss posture: inspect, verify, don't trust
- `WORKFLOW.md` — the concrete step sequence for a delegated task
- `TOOLS_AND_MCP.md` — each CLI's real invocation path, known quirks, when to use which
- `PROMPT_ARCHITECTURE.md` — how to write a delegation prompt that gets real work back
- `CONTEXT_MODEL.md` — what state carries across delegated calls, what doesn't
- `QUALITY_GATES.md` — the checks a delegated result must pass before you act on it
- `FAILURE_AND_RECOVERY.md` — known failure modes (hangs, stale claims, hallucination) and the fix for each
- `TEST_SUITE.md` — how this skill itself was validated, and how to re-validate it
- `EVOLUTION_PLAN.md` — what's deliberately out of scope for v1, and the trigger to revisit

## Quick start

1. Read `WORKFLOW.md` for the step sequence.
2. Read `TOOLS_AND_MCP.md` for the resolution order (bare command first, full path as fallback) and the exact fallback path for whichever CLI you're about to shell out to — all four have hit PATH gaps in the past.
3. Before writing the delegation prompt, skim `PROMPT_ARCHITECTURE.md` — a vague prompt gets a vague, unverifiable answer back.
4. After the result comes back, do not report it to the user until it has passed `QUALITY_GATES.md`.

## The one-sentence version

Claude Code is the coordinator, not a participant: applying changes and the coordinator's own verification checks stay here (see `REQUIREMENTS.md` R1); the actual diagnostic/analytical legwork — why is this happening, what's the root cause, deep research, adversarial review, large builds — defaults to delegation. Nothing delegated gets reported as fact until independently checked against real evidence — a subagent's claim is a hypothesis, not a result.
