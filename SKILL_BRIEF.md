# SKILL_BRIEF

## What this is

A standing operating posture for any task the user tags "use multi AI framework" (or equivalent — "deep work", "heavy work", "use all AIs"). It is not a single tool call; it's a way of running a whole task across multiple independently-billed AI CLIs already installed on this machine (Claude Code, Hermes Agent, OpenAI Codex CLI, OpenCode, Google Antigravity), coordinated by Claude Code.

## Why it exists

Two problems it solves, discovered the hard way across real sessions:

1. **Claude Code's own token budget is finite and billed to the user's plan; the other four CLIs are on separate subscriptions/credits.** Routing heavy research or review work through them is free-er and often produces a genuinely different, useful second opinion — but only if the result is actually checked, not rubber-stamped.
2. **A single AI's claim — including this one's own — is not evidence.** Across real sessions this framework has caught: a research agent inventing a plausible-sounding but wrong root-cause (storage-fullness "causing" a specific load average, unproven), a code reviewer flagging real bugs mixed with two confidently-stated false ones that contradicted its own earlier verified research, and a delegated pricing lookup that needed re-verification after a shell-escaping bug silently mangled the numbers in the prompt. Every one of those was caught only because the coordinator checked before reporting, not because the delegate was reliable.

## One-paragraph mental model

Think of Claude Code as a hardcore engineering lead running a small team of contractors (Hermes, Codex, OpenCode, Antigravity). The lead doesn't do the contractors' typing for them, but also doesn't take their word for anything that can be checked — a contractor's report gets read, cross-referenced against the actual system/file/API, and only then relayed to the client (the user) as fact. Revised 2026-07-29 (see `EVOLUTION_PLAN.md`): the lead is not the primary investigator either — diagnosis defaults to delegation, per `REQUIREMENTS.md` R1 and `WORKFLOW.md` step 2. What stays with the lead personally is applying real changes (the lead is the only one with hands on this actual machine for mutation) and the targeted confirmation checks that verify a delegate's specific claim — not the broad investigative legwork that produces the claim in the first place.

## Non-goals

This is not an excuse to fan out work in parallel for speed. Two different things both true at once, not a contradiction: (1) the coordinator's own operating instructions say "one at a time, sequential" specifically for the `Agent` tool (a system-prompt-level instruction, not CLAUDE.md — checked directly during this skill's own review: neither the project-level nor global CLAUDE.md contains any concurrency directive); (2) dispatching a background shell call to an external CLI (`run_in_background: true`, per `WORKFLOW.md` step 4) and continuing other work while it runs is normal and expected — that's not "fanning out for speed," it's not blocking on I/O. What's actually out of scope is racing multiple CLIs against the *same* question with no reconciliation process — a few independent background calls running concurrently on *different* pieces of work is the normal pattern, not an exception to it. If genuine same-question racing ever becomes the goal, `QUALITY_GATES.md` Gate 5 already defines the reconciliation method: resolve via primary source, don't average. It is also not a substitute for the user's own judgment on irreversible actions — delegation changes who does the analysis, not who authorizes a stop/start on a production VPS or a machine-type change with real cost.
