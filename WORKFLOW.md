# WORKFLOW

The concrete step sequence for a task tagged "use multi AI framework". Not every step applies to every task — scale to what's actually being asked.

## 1. Scope and split

Before delegating anything, work out: what tiny part needs a live local look so a delegate can be briefed or verified (stays local), what substantive work needs deep research, diagnosis, drafting, or a from-scratch build plan (delegates), what part needs an adversarial second opinion on something already produced (delegates to a *different* tool than whatever produced it).

## 2. Baseline evidence: a quick local look, then delegate the actual investigation

The coordinator is the boss, not the primary investigator or primary worker — per real evaluated evidence (see `EVOLUTION_PLAN.md`'s note on the 2026-07-29 research pass), letting the lead do all the diagnostic legwork itself is exactly the "heavy lifting" pattern this framework exists to avoid. The 2026-08-04 standing correction makes this stricter: Claude Code should minimize its own direct substantive work because its usage budget burns too fast. The right split:

- **Do a fast, minimal local check first** — just enough to give a delegate real evidence instead of a vague symptom ("app X isn't in Windows Search" → confirm the binary still exists on disk, that's a 5-second check). Do NOT do the full root-cause investigation, broad file reading, or first-pass drafting yourself.
- **Delegate the actual work** — "why would this happen, what's the likely cause, what commands would confirm and fix it, what should the draft/plan/code look like" — to a worker CLI, with the quick local check's findings included as evidence.
- **The coordinator then verifies and executes** the delegate's proposed fix directly (see step 8) — diagnosis delegates, mutation and verification stay centralized. This isn't a contradiction: a delegate can tell you *what* is probably wrong and *what command* would fix it; only the coordinator has the actual system access to safely run that command and confirm it worked.

Exception: if the check is trivially fast (one command, unambiguous result — e.g. "does this file exist") and delegating would cost more round-trip time than it saves, just do it. Use judgment, but default to delegating the *investigation*, not just the research.

**Where "verifying the delegate's diagnosis" stops and "redoing the diagnosis" starts:** the delegate owns hypothesis generation and broad diagnostic exploration (log-diving, multi-file inspection, trying several theories). The coordinator's verification (step 6) is one or two *targeted* confirmation checks against the specific claim the delegate made — running the specific command it named, reading the specific file/line it cited. If confirming the claim turns out to require its own broad exploration (the delegate's proposed check doesn't actually confirm it, or raises a new question), that new evidence goes back to a delegate rather than the coordinator quietly continuing the investigation solo — that slide is exactly the "heavy lifting" this step exists to prevent.

## 3. Write the delegation prompt

See `PROMPT_ARCHITECTURE.md`. Include the real evidence from step 2 inline. State explicitly that the result will be fact-checked — the observed pattern across this session is that delegates hedge more appropriately instead of overclaiming when told this, though it hasn't been measured with a controlled comparison.

## 4. Dispatch

Follow `TOOLS_AND_MCP.md`'s resolution order — try the bare command first, fall back to the full path listed there if it's not found (a shell opened before the 2026-07-29 PATH fix still needs the full path until restarted; see `FAILURE_AND_RECOVERY.md` for the underlying cause). Run in background (`run_in_background: true`) for anything that isn't a quick lookup, so the coordinator can keep working. Do not fabricate or predict the result while it's in flight if asked for status — report what's actually known.

## 5. Read the full result, don't skim for the summary line

Delegates sometimes bury the important caveat ("I'm not confident about X") in the middle of a long report. Read it fully before deciding what to verify.

## 6. Verify per QUALITY_GATES.md

Every claim that will be acted on or repeated to the user gets checked against a primary source before it's trusted.

## 7. If verification finds an error, diagnose it, then hand off to step 8a

A single wrong claim doesn't invalidate the whole report — first check whether it was a fluke on the coordinator's own side (e.g. a shell-escaping bug mangling numbers before the delegate ever saw them, per `FAILURE_AND_RECOVERY.md`). If it's a coordinator-side fluke, just fix the prompt and re-send — that's not a delegate failure. If it's a genuine delegate mistake, this is a step 6 failure: it enters the step 8a loop (reassign with the specific failure context, escalate to a different tool if it fails twice) rather than being resolved ad hoc here. Step 8a's retry/escalate/stop sequence is the *only* failure-handling algorithm in this workflow — it covers failures surfaced at step 6 and failures surfaced at step 8, not two different processes.

## 8. Apply, test, re-verify — the coordinator holds mutation authority

If the task involves making a change (code, config, infra), **the coordinator applies it, not the delegate** — even when a delegate proposed or drafted the fix. Real evidence backs this specifically (not just an accountability preference): production multi-agent research found that letting agents act unsupervised increases error amplification several-fold over centralized execution. A delegate can hand back a diff, a command, a config value; the coordinator reviews it, runs it, and re-verifies against real evidence that it worked — the same standard as step 6, applied to the coordinator's own action now, not just the delegate's claim.

## 8a. Loop instead of accepting a weak result

Any genuine delegate failure — caught at step 6 (a claim doesn't check out) or step 8 (an applied fix doesn't re-verify) — enters this same loop; there is one failure-handling algorithm, not two. If a delegate's output was too vague/wrong to act on, don't silently patch it up solo and move on — that's back to doing the heavy lifting yourself. Loop:

1. **Reassign with more context** — the specific way it failed, any new evidence gathered — to the same delegate, once.
2. **If it fails again, escalate to a different tool** — a different CLI, or a different role (e.g. send it to an adversarial-review prompt instead of another research prompt) rather than repeating the same request a third time.
3. **Set a stopping condition up front** — for any task where getting it wrong has a real cost (it feeds a decision, a fix, or something reported to the user as fact — as opposed to, say, a throwaway exploratory question), decide before starting how many loop iterations are reasonable (two or three is typical) so this doesn't run away. If still unresolved after that: "do the final piece directly" means resolving the *specific narrow claim* that failed verification (e.g., confirm one specific fact via one direct check) — it does not mean quietly absorbing the whole original diagnostic task the delegate was assigned. If what's left after the loop is still a broad investigation rather than one narrow unresolved claim, that's a signal to report the honest state of things to the user (what's confirmed, what isn't, and that two delegate attempts didn't resolve it) rather than have the coordinator finish the investigation solo.

This loop — assign, verify, reassign-or-escalate — is the concrete mechanism behind "monitor work that's failing and reassign it," not just a phrase.

## 9. Report honestly

State what's confirmed, what's still uncertain, and what (if anything) the delegate got wrong and how that was caught — this is useful information for the user, not something to smooth over.
