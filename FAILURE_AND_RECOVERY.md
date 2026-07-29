# FAILURE_AND_RECOVERY

Known failure modes, each one observed for real in this environment, with the actual fix.

## Delegate process hangs indefinitely with no output
**Seen with:** Hermes (30+ min, zero output, no error), Codex (stuck on "Reading additional input from stdin...").
**Fix:** don't wait indefinitely, but don't use a single fixed timer either — anchor the wait to the task's own nature: a quick fact lookup or single-file review producing zero output after ~2-3 minutes is suspicious; a task that legitimately involves several web searches or a large codebase scan can reasonably run 5-10 minutes before it's worth checking. Whatever the anchor, check the process is still alive (`ps aux | grep <tool>`) before killing anything — "no output yet" plus "process alive" together aren't enough to call it hung; "no output" plus "process gone/zombied" is. If it looks genuinely stuck rather than just slow, kill it (`kill -9 <pid>`) and either retry with the stdin-piping fix (`echo "" | codex exec ...`) or switch to a different tool for that task.

## Bare command not found despite being "installed"
**Seen with:** `opencode`, `agy`, `graphify`, `codex`, `hermes` all at different points.
**Root cause (confirmed, not guessed):** the directories containing these binaries' shims (`.npm-global`, `agy\bin`, `.local\bin`) were absent from the persistent Windows PATH — checked directly via `[Environment]::GetEnvironmentVariable("Path","User"/"Machine")`, confirmed empty. Not a broken install.
**Fix:** use the full path from `TOOLS_AND_MCP.md` immediately; separately, add the missing directory to the persistent User PATH so future *new* shells resolve the bare command (does not retroactively fix already-running shells — those need a restart).
**Lesson:** before concluding a tool is "broken" or "not being used", check PATH resolution directly rather than assuming the install itself failed.

## A delegate's confident claim turns out to be wrong
**Seen with:** a code reviewer asserting two schema keys were wrong, when direct inspection of real data showed the reviewer's own earlier research (confirming those exact keys) was correct and this later claim wasn't; a reviewer flagging a path-traversal risk that direct testing showed was already mitigated by existing code.
**Fix:** per `QUALITY_GATES.md` Gate 2 and 4 — check every individual finding against real data before acting on it, especially when it contradicts something already verified.

## A number looks wrong after review, but the research wasn't actually wrong
**Seen with:** a prompt containing literal `$196`/`$132` passed through a Bash-invoked shell got the numbers mangled by shell variable expansion before the delegate reviewing it ever saw the real figures — the reviewer correctly flagged the *mangled* numbers as inconsistent with official pricing, which read as "the original research was wrong" when actually the original research (from a separate, earlier call) had the correct numbers all along.
**Fix — behavioral (fallback):** escape or avoid literal shell-special characters (`$`, backticks, unescaped quotes) in delegation prompts.
**Fix — structural (preferred, do this instead when the prompt contains numbers/code/anything with `$`, backticks, or quotes):** write the prompt to a temp file first (`Write` tool, plain text, no shell involved in creating it), then pass it to the delegate via stdin redirection or a `--file`-style flag if the CLI supports one, rather than as an inline shell argument. This removes the shell-interpolation step entirely instead of relying on remembering to escape every special character correctly. When a review flags a number as wrong, also check whether the number that reached the reviewer was actually the number originally researched, in case this class of bug recurs before the structural fix is adopted everywhere.

## Recommended plan omits real cost of executing it
**Seen with:** a scaling recommendation that didn't mention the machine-type change requires a full instance stop/start (real downtime for 20 running containers).
**Fix:** per `QUALITY_GATES.md` Gate 6 — explicitly ask delegates for the operational cost of executing their recommendation, and independently sanity-check this against known platform behavior before presenting the plan.

## A delegate's own internal skill/routing system hijacks the task
**Seen with:** Codex CLI has its own bundled "brainstorming" skill that auto-triggers on prompts sounding like "design/build a new feature" (even a research prompt that mentioned "real code changes needed... will be applied directly to the file" was enough to trigger it). It then followed its own protocol — asked whether to enable an interactive "visual companion" browser tool and stopped there, since a non-interactive `codex exec` call has no way to answer a follow-up question. The task never completed; the "result" was just the stalled question.
**Fix:** when delegating to a tool with its own skill/routing system, phrase the prompt to avoid the trigger pattern for research-only work — explicitly state "this is a factual research question, not a new feature design, answer directly without asking questions or offering interactive tooling, this is a non-interactive headless call that cannot receive follow-up answers." If a delegate call returns a question instead of a result, that's the signal this happened — don't treat the question as the deliverable, retry with clearer framing.

## A delegation fails verification and the failure pattern recurs across tasks
**Seen with:** not yet observed as a recurring pattern for a specific tool/task-type combination — this entry exists so the next occurrence gets logged here rather than being silently absorbed.
**Fix:** per `WORKFLOW.md` step 8a / `REQUIREMENTS.md` R9 — reassign once with the specific failure context, escalate to a different tool if it fails again, stop and do the final piece directly (saying so) if still unresolved after that. If the *same tool* fails on the *same claim type* across more than one real incident, add a specific entry here documenting the pattern (per `EVOLUTION_PLAN.md`'s scoring-system revisit trigger) rather than treating each occurrence as a one-off.

## A GCP (or similar platform) operation fails partway through a multi-step infra change
**Seen with:** attempting to shrink a persistent disk via snapshot+recreate — GCP hard-rejects a disk smaller than the snapshot's source disk size, discovered only by attempting it.
**Fix:** when a plan's feasibility depends on undocumented-to-the-coordinator platform behavior, either verify the constraint via the platform's own docs before attempting a stateful step (like stopping a production instance), or if attempting is the only way to find out, minimize the exposure window (revert/restart immediately on failure) and report the real constraint found rather than continuing to push against it.
