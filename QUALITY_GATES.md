# QUALITY_GATES

Checks a delegated result must pass before the coordinator acts on it or reports it to the user as fact. Not every gate applies to every kind of result — apply the ones relevant to what's being checked.

## Gate 1 — Primary-source check, for any number or fact that drives a decision
Pricing, benchmark figures, counts, schema claims: re-derive from the actual primary source (official page, live system, real file) rather than trusting the delegate's restatement. Per `REQUIREMENTS.md` R3, two delegates agreeing is not by itself a pass — correlated failure across models is real (see `EVOLUTION_PLAN.md`'s 2026-07-29 entry) and agreement can mean two models sharing the same wrong answer, not two independent confirmations. Two lookups agreeing is only a pass when each one independently touched the actual primary source (e.g., two different delegates each fetched the real pricing page directly) — not when both are just restating the same figure from memory or from each other. If the delegate cites a source, fetch it directly rather than trusting the citation is accurate.

## Gate 2 — Schema/API claims checked against real live data, not just documentation
General research about "how X's format works" can be correct in general and still wrong about this specific instance's actual behavior. Pull one real example and check the claim against it directly before trusting it.

## Gate 3 — "It works" claims need an artifact, not a description
A build succeeding, a test suite passing, a process not crashing — each of these is evidence for a narrower claim than "the feature works." Before reporting a fix or feature as done, there is a concrete result attached: real command output, a re-run reproduction of the original failure now succeeding, or (for UI) an actual launch-and-observe, not just a compile.

## Gate 4 — Adversarial review findings get individually re-checked, not batch-trusted
A review that finds N issues is not "N real issues" — each one gets checked against the actual code/data before being acted on. A real review pass in this framework's own history produced 7 findings, of which independent re-checking confirmed some as genuine and rejected others as directly contradicting verifiable facts (the reviewer's own earlier confirmed research, and the actual regex behavior on inspection) — batch-trusting would have introduced new bugs while "fixing" things that weren't broken. The exact split isn't preserved from that incident; the lesson that matters is re-check each one, not the specific count.

## Gate 5 — Contradictions between delegates get resolved, not averaged
If two delegates disagree, or a delegate disagrees with something the coordinator can check directly, investigate which one is right using a primary source — don't split the difference or silently pick one.

## Gate 6 — Cost/downtime/risk claims get stated plainly, not smoothed over
If applying a recommended change means real downtime, real spend, or is irreversible, that fact is surfaced explicitly before acting — a delegate's plan that omits this (has happened: a resize recommendation that didn't mention it requires a full stop/start) gets that gap filled in by the coordinator, not passed through silently.

## Gate 7 — Uncertainty is stated as uncertainty
If a claim could not be verified in the time available, it's reported with that caveat attached — not upgraded to the same confidence as a verified claim for the sake of a cleaner-sounding summary.
