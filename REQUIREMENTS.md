# REQUIREMENTS

What invoking this framework actually obligates the coordinator (Claude Code) to do — not aspirational. `TEST_SUITE.md` validates the subset of these that are checkable via retrospective incident tracking and its self-check checklist. R1 has real incident evidence behind it (the user-flagged pattern of over-centralized diagnosis, see `EVOLUTION_PLAN.md`). R9 has a checklist item but, as of this writing, no real incident has yet occurred to validate it against — it's a rule stated in advance of evidence, not one confirmed by a real failure-and-recovery case yet (see `FAILURE_AND_RECOVERY.md`'s placeholder entry). It does not yet mechanically verify all nine requirements, and its own known limitation says the checklist is advisory, not enforced.

## R1 — Execution and verification stay local; investigation defaults to delegation
Revised 2026-07-29 after real evidence review (see `EVOLUTION_PLAN.md`) — the original version of this rule had the coordinator doing too much of the diagnostic work itself, which is exactly the "heavy lifting" this framework exists to keep off the coordinator's plate. The actual split:
- **Stays local, no exception:** running the command that changes something (an edit, an install, a config write, an infra action), and confirming afterward that it actually worked. This is deliberate, not a convenience choice — production multi-agent research shows unsupervised delegate execution measurably increases error amplification. Only the coordinator has real accountability for what it claims to have verified.
- **A fast, minimal check to generate real evidence** (does this file exist, what does this one log line say) — quick enough that delegating would cost more round-trip time than it saves.
- **Defaults to delegation:** the actual diagnostic legwork — why is this happening, what's the likely root cause, what would fix it — once the coordinator has enough of a starting fact to hand a delegate real evidence instead of a vague symptom. See `WORKFLOW.md` step 2 for the concrete sequence.

## R2 — Analytical/generative work delegates
Deep research (multi-source, needs real web search), adversarial code/plan review, large from-scratch content generation (a doc suite, a big research report) routes to one of Hermes/Codex/OpenCode/Antigravity. Default assignment (not exclusive — match to what's actually needed; this is a starting point, not a lane each tool is boxed into):
- **Hermes** — deep research with its own bundled skill set. Lean external-facing (pricing, current docs, general research) — this default hasn't been specifically stress-tested against internal/system work, it's a starting assumption based on its research-skill bundle, not a measured result.
- **Codex** — code review, second implementation opinion, adversarial "find the holes" passes. It has also been used for internal/system-adjacent review this session (e.g. this very adversarial-review pass) with real, actionable results — don't over-narrow this one to "external code only," but "equally strong" as external research isn't a claim actually measured here, just an observation that it hasn't shown a weakness there yet.
- **OpenCode** — cheaper/alternate-model-family research and review, second opinions. Demonstrated real strength once, reading this machine's own local files directly to reverse-engineer a schema rather than guessing from general knowledge — one confirmed data point, not yet a broad pattern; lean internal/system-facing when that's the need and keep testing it.
- **Antigravity** — research benefiting from its own agent harness/tools, large report generation, and (based on exactly one verified test task, not a general capability yet) drafting real system/coding work (writing code, proposing commands) for the coordinator to review and apply. Verified 2026-07-29 as an isolated capability benchmark, not a mutation-policy change, and not yet a broad claim: given a real task (write a Python script reading live process memory via psutil, plus a pytest suite, then actually run both, in Antigravity's own sandboxed scratch directory), it installed its own dependencies, wrote correct working code unprompted for the exact spec, and its self-reported command output matched an independent coordinator re-run on the test results (5/5 passed) and structurally on the live process listing. **This does not relax R1** — in normal delegated work Antigravity (like every other delegate) returns code/diffs/commands for the coordinator to review and apply, it does not get standing write access to the coordinator's own working files. What the test establishes is real capability on one Python system-inspection task, not a broad "Antigravity can be trusted with system/coding work generally" claim — treat further task types as unverified until each is actually tried.

## R3 — Every delegated claim is verified before being reported as fact
No exceptions. "Verified" means one of: re-derived from a primary source the coordinator checked directly (a file, a live command, an official pricing page), or cross-checked against a second independent delegate with the contradiction resolved via a primary source (not just agreement-counting). "The coordinator already knows this from training data" does NOT count as verification — that's exactly the kind of unverified claim this framework exists to distrust, whether it comes from a delegate or from the coordinator's own prior belief. The only exception is something the coordinator has personally confirmed with real evidence *earlier in this same session* (e.g., a fact already established via a live command run a few steps ago) — that's reuse of already-verified evidence, not a new unverified claim. A delegate's assertion alone, however confident-sounding, does not clear this bar — and neither does the coordinator's own unconfirmed assertion.

## R4 — Corrections get surfaced, not silently applied
If a delegate's finding contradicts an earlier delegate's finding, or contradicts something the coordinator can check directly, the contradiction is investigated and the resolution (which one was right, and why) is stated — not quietly picked one way with no explanation.

## R5 — Real evidence accompanies every "it works" claim
A build succeeding is not evidence a feature works. A test passing is evidence for what that test covers, nothing more. Before declaring a fix or feature done, there is a concrete artifact — a command output, a screenshot, a re-run reproduction — attached to the claim.

## R6 — Irreversible or costly actions still require the user
Delegation changes who does the analysis and drafting. It never changes who authorizes a production stop/start, a spend-affecting resize, a destructive delete, or anything else already gated by the standing safety rules. A subagent recommending an action is not authorization to take it.

## R7 — Background delegation is reported honestly while in flight
If asked for status mid-delegation, the coordinator reports what it actually knows (still running, here's the last real checkpoint) — never a fabricated or predicted result.

## R8 — Not everything analytical automatically delegates
**This table applies to R2 (analysis/review/generation) only — it does not govern R1 (diagnosis).** Diagnosis has its own, narrower exception (R1: one trivially-fast, unambiguous check) and is not reopened by any row below, including the critical-path one. R2's "delegate the thinking-heavy work" is not unconditional for analytical work. A quick decision matrix:

| Signal | Lean toward |
|---|---|
| Needs current/real-time info (pricing, live docs, current model availability) the coordinator can't verify from what it already has | Delegate (a tool with real web search) |
| Small, well-scoped review of a diff/file the coordinator already has fully in context | Do it inline — shelling out plus verifying often costs more than just doing it |
| Large from-scratch generation (a doc suite, a big report) where breadth matters more than the coordinator's own context budget | Delegate |
| A second, genuinely independent opinion is the point (adversarial review) | Delegate — self-review isn't independent |
| A piece of *analysis* (not diagnosis) is on the critical path and a delegate hang would block progress with nothing else useful to do meanwhile | Consider doing that analysis inline, or delegate with a tight mental timeout per FAILURE_AND_RECOVERY.md — this row never applies to root-cause diagnosis, see above |

When genuinely unsure, the cost of delegating-and-verifying is usually still lower than the cost of a wrong inline answer — but "always delegate" isn't the rule, "delegate when it's actually the better tool for this specific piece" is.

## R9 — Failing work gets looped, not silently absorbed
When a delegate's result fails verification (R3) or doesn't hold up, the coordinator's job is to manage that failure as a loop — reassign with the specific failure context, escalate to a different tool if it fails twice, set a stopping condition up front — not to quietly finish the work itself and move on. See `WORKFLOW.md` step 8a. Silently absorbing a failed delegation back into direct coordinator work defeats the point of delegating in the first place and hides a real signal (this delegate/this prompt/this task type isn't working) that `FAILURE_AND_RECOVERY.md` should be capturing.
