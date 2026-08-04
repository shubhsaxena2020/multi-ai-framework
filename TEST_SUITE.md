# TEST_SUITE

This skill is a behavior specification, not code — it can't be unit-tested in the traditional sense. What it can be checked against is the real session history that produced it, and a self-check checklist to run on each future invocation.

## Retrospective validation — did the documented behavior actually happen?

Every claim in `BEHAVIOR_SPEC.md` and `FAILURE_AND_RECOVERY.md` traces to a real, specific incident in this environment's actual session history, not a hypothetical:

| Claim | Real incident it's based on |
|---|---|
| Delegate schema claims need checking against live data | A code reviewer asserted two JSON keys were wrong; direct inspection of a real transcript file confirmed the original code (and the reviewer's own earlier research) was right, the later claim wrong |
| Hangs happen and need active handling, not indefinite waiting | Hermes hung 30+ min with zero output on a pricing-research task; Codex separately hung on stdin on a different task |
| PATH gaps look like "tool is broken" but usually aren't | `opencode`/`agy`/`graphify` all resolved to "command not found" system-wide; confirmed via direct registry-level PATH inspection that the install was fine, the PATH entry was simply never added |
| Shell-escaping can silently corrupt a number before a delegate ever sees it | A literal `$196`/`$132` in a Bash-invoked prompt got mangled, producing a false "the pricing research was wrong" signal in a later review pass |
| A recommended plan can omit real execution cost | A VPS resize recommendation didn't mention it needs a full stop/start; caught before the user was told it'd be free/instant |
| Platform constraints can invalidate a plan only discoverable by attempting it | A disk-shrink-via-snapshot plan looked sound until GCP hard-rejected it at execution time — not documented anywhere the coordinator checked beforehand |
| The coordinator doing its own diagnostic legwork is the "heavy lifting" this framework exists to avoid, not a neutral default | User directly flagged this pattern (VPS log-diving, registry/root-cause work done solo) after a full session of it; two independent delegated research passes (Codex, Antigravity), with the two most load-bearing citations HTTP-verified directly, confirmed centralized-diagnosis-by-the-lead is the wrong shape versus production frameworks — see `EVOLUTION_PLAN.md`'s 2026-07-29 entry |
| Agreement between delegates isn't a substitute for verification | Same research pass surfaced correlated cross-model failure rates of 42-60% on trick/ambiguous questions — directly informed rejecting "agents confirm each other's work" as a reliability mechanism rather than just a style preference |

This is not an exhaustive list of every incident, but every entry is a real one, not an invented example.

## Self-check checklist — run this on each invocation of the framework

- [ ] Did any number, fact, or schema claim that will be acted on get checked against a primary source, not just relayed from a delegate?
- [ ] Does the report to the user distinguish "confirmed working" (has an artifact) from "should work" (compiled/ran without crashing) from "researched, not yet built"?
- [ ] If a delegate's finding was rejected or corrected, is that stated to the user rather than silently smoothed over?
- [ ] If the task involves any downtime, cost, or irreversible step, was that made explicit before or during execution, not discovered after?
- [ ] If a delegate recommended a stop/start, a spend-affecting change, a destructive delete, or any other action already gated by the standing safety rules — did the coordinator get the user's actual authorization before executing it, rather than treating the delegate's recommendation itself as sufficient? (R6 — this is a safety rule, treat a miss here as more serious than a miss on any other checklist item)
- [ ] If asked for status on an in-flight delegation, did the coordinator report only what it actually knew at that moment, rather than a fabricated or predicted outcome? (R7)
- [ ] If any background/delegated task ran for more than a couple of minutes, did the coordinator perform real liveness checks roughly every 2 minutes instead of waiting passively? (R7, `FAILURE_AND_RECOVERY.md`)
- [ ] If verification failed, was it actually a delegate mistake — or a coordinator-side fluke (e.g. a shell-escaping bug mangling the prompt) that just needs a corrected re-send, not a strike against the delegate? (`WORKFLOW.md` step 7)
- [ ] Was the actual change (edit, install, config write) applied and re-verified by the coordinator itself — not just relayed from a delegate's *description* of having done it? (R1)
- [ ] Did diagnostic/investigative work default to delegation rather than the coordinator doing the root-cause legwork solo, outside the narrow one-command exception? (R1, `WORKFLOW.md` step 2)
- [ ] If a delegation failed verification, did it go through the reassign-once → escalate-to-a-different-tool → stop loop, instead of being silently finished solo? (R9, `WORKFLOW.md` step 8a)

## Known limitation of this validation approach

Because the "test suite" is retrospective incident tracking rather than executable tests, it can't catch failure modes that haven't happened yet in this environment. Treat `FAILURE_AND_RECOVERY.md` as a living document — add a new entry the next time a delegate's output turns out wrong in a new way, rather than assuming the existing list is complete.

**A second, sharper limitation, surfaced by this skill's own review process:** the self-check checklist above is advisory, not enforced. Nothing prevents a future invocation from skipping every verification step while still mentally "checking every box." The only real safeguard is that this skill's own writing gets re-reviewed periodically the same way it was built — by sending it to a different tool and asking it to find holes, not by trusting the coordinator's self-assessment. If this skill is ever revised, run it through `PROMPT_ARCHITECTURE.md`'s adversarial-review pattern again rather than assuming a self-edit is safe.
