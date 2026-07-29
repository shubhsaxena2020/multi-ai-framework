# CONTEXT_MODEL

What state actually carries across a delegated call, and what doesn't — getting this wrong produces prompts that assume shared context the delegate doesn't have.

## What does NOT carry over automatically

- **Nothing from the current Claude Code conversation.** Each `hermes`/`codex`/`opencode`/`agy` invocation is a cold process with no memory of anything said in this session unless it's explicitly included in the prompt. A delegation prompt is self-contained by necessity — assume zero shared context.
- **Prior delegated calls to the same tool, in a different invocation.** Unless using an explicit resume/continue mechanism (Codex has `resume`/`fork`; check whether the specific tool supports it before assuming continuity), each call starts fresh.
- **The coordinator's own reasoning about why a task matters.** If it's relevant to how the delegate should approach the work, it has to be stated in the prompt, not implied.

## What DOES carry over, and is worth reusing rather than re-deriving

- **Files already on disk.** A delegate with file-read tools (OpenCode, Codex, Antigravity all have these) can inspect real files directly — point it at the actual path rather than pasting a summary, when the file is the primary source of truth (a config, a script under review, a real transcript).
- **This skill's own docs.** `TOOLS_AND_MCP.md` and `PROMPT_ARCHITECTURE.md` are reusable across tasks — read them once per task, not re-derive the binary paths or prompt structure from scratch each time.
- **Memory files** (`%USERPROFILE%\.claude\projects\*\memory\`) — durable facts about the user's systems, established once, that a delegation prompt can reference or the coordinator can pull from before writing the prompt.

## Practical implication

Before writing a delegation prompt, decide explicitly what the delegate needs to know that it cannot infer: the real numbers already gathered (per `WORKFLOW.md` step 2), the specific system/decision this feeds into, and any constraint that would otherwise be invisible to a cold-start process (e.g., "don't recommend X, we already ruled that out because Y" — if this isn't stated, the delegate has no way to know it and may re-propose X).
