# PROMPT_ARCHITECTURE

How to write a delegation prompt that comes back verifiable, not just plausible-sounding.

## Structure that has worked in practice

1. **State the real context and constraints up front** — what system/file/decision this feeds into, so the delegate optimizes for the actual use case rather than a generic answer.
2. **Include real evidence inline**, not a paraphrase of it. Paste actual log lines, actual config values, actual numbers already gathered locally (per `WORKFLOW.md` step 2). A delegate reasoning from real data produces a checkable answer; a delegate reasoning from a vague description produces a guess dressed as an answer.
3. **Ask numbered, specific questions**, not "tell me about X". Vague prompts get vague, hard-to-verify answers back.
4. **State explicitly that the result will be fact-checked.** This is not a formality — the observed pattern (not a controlled measurement) is that delegates respond with more hedged, source-cited answers instead of confident guesses when told this upfront. Phrasing used successfully: *"this will be reviewed adversarially and fact-checked before we act on it, so be precise and cite what you found, don't guess."*
5. **Ask for sources explicitly**, and prefer delegates with real web search (Codex, Hermes, Antigravity, OpenCode all have it) over ones reasoning from training data alone when the question is time-sensitive (pricing, current model availability, current library versions).
6. **Ask the delegate to flag its own uncertainty** rather than presenting everything at uniform confidence — "flag anything you're not confident about rather than guessing" has reliably produced useful caveats in practice.

## For adversarial review specifically

- Explicitly tell the delegate its job is to find holes, not confirm — "don't be agreeable, actively look for problems" is the phrasing that has reliably shifted a delegate from validation-mode to critique-mode in practice (not a controlled A/B result, just the repeated observation).
- Give it the claim under review verbatim plus the evidence backing it, and ask for a verdict per claim (confirmed / needs caveat / reject) rather than open-ended commentary — this produces a result that's directly actionable in `QUALITY_GATES.md` rather than another paragraph to interpret.
- Send the review to a *different* tool than whatever produced the thing under review — self-review from the same model/tool has less independence.
- **What a full review pass looks like in practice:** one delegate reads every file/claim in scope in full (not a sample) and returns concrete file:line-cited findings, categorized by severity; the coordinator then independently re-checks each finding against the actual code/data (`QUALITY_GATES.md` Gate 4) before applying any fix. For higher-stakes reviews (e.g. before publishing something publicly), a second pass from a *different* delegate than the first catches what the first one's blind spots missed — this has happened in practice: a second reviewer found real issues (unfalsifiable claims, contradictions) the first reviewer didn't flag. There's no fixed number of passes required; use judgment on how many independent looks the stakes justify.

## Anti-patterns (caused real problems this session)

- **Dollar signs and other shell-special characters inside a prompt string passed through Bash** get mangled by shell expansion before the delegate ever sees them (a `$196` became `$96` in one real prompt, then a downstream reviewer correctly flagged the *mangled* number as wrong, which read like the original research was wrong when it wasn't). Escape or avoid literal `$`, backticks, and unescaped quotes in delegation prompts sent via a shell.
- **Asking a delegate to review its own earlier output** produces confirmation, not adversarial checking — route review to a different tool.
- **Padding a prompt with speculative "and also consider..." tangents** dilutes focus and produces a less useful, harder-to-verify answer than a tightly scoped one.
