# TOOLS_AND_MCP

**Platform note:** this file documents one real Windows setup (paths, `.cmd` shims, PowerShell quoting quirks) as a concrete worked example, not a universal spec. On Linux/macOS the binaries, shell (`ps aux` in `FAILURE_AND_RECOVERY.md` is Unix-native and will work there as-is), and path conventions will differ — replace the specifics below with your own tools' real invocation details; the pattern (bare-command-first resolution order, documenting known hangs, real config locations) is what's portable.

Real, verified binary locations and quirks for each CLI, as of 2026-07-29. `.npm-global` and `.local\bin` were added to the persistent User PATH on this date, so bare commands (`opencode`, `codex`, `claude`, `graphify`) should resolve in *new* shells — but any Claude Code session/terminal opened before that fix still needs the full path until restarted.

**Resolution order:** try the bare command first (cheap, and confirms whether the PATH fix has propagated to this shell). If it fails with "not found", fall back to the full path listed below. If a path below has gone stale (e.g. Codex's hash-suffixed folder changed after an update), glob for the binary rather than giving up — e.g. `Get-ChildItem -Path "$env:LOCALAPPDATA\OpenAI\Codex\bin" -Filter codex.exe -Recurse`. Don't default straight to the full path on every call; that's guaranteed to rot as installs update, and bare-first costs nothing when it works.

## Hermes Agent
- Binary: `%USERPROFILE%\AppData\Local\hermes\hermes-agent\venv\Scripts\hermes.exe`
- Invoke: `hermes -z "<prompt>"`
- Config: `%LOCALAPPDATA%\hermes\config.yaml` (custom_providers, model.provider must be the literal provider name — `model.provider: auto` silently skips custom provider resolution entirely)
- **Known failure mode:** has hung indefinitely with zero output on at least two real occasions (30+ min, no error, no output file content). Per `FAILURE_AND_RECOVERY.md`'s timing guidance, kill it (`kill -9 <pid>`) and retry, or switch to Codex/OpenCode, once it's clearly past the reasonable window for the task's own nature and the process check shows it isn't making progress.

## OpenAI Codex CLI
- Binary: `%USERPROFILE%\AppData\Local\OpenAI\Codex\bin\<hash>\codex.exe` — the hash folder changes on update; glob for `codex.exe` under `AppData\Local\OpenAI\Codex\bin\` if the known path stops working. Also available via the npm shim at `%USERPROFILE%\.npm-global\codex.cmd` once that's on PATH.
- Invoke: `codex exec --skip-git-repo-check "<prompt>"`
- Config: `%USERPROFILE%\.codex\config.toml` — one real example had `model = "gpt-5.6-sol"` set on 2026-07-29. **Don't copy that model string as-is** — model availability changes; whatever you set, verify the model string is still valid by testing with `-m <model>` before committing it to config, don't just guess a name.
- **Known failure mode:** occasionally hangs on "Reading additional input from stdin..." and never proceeds. Fix: pipe empty stdin explicitly — `echo "" | codex exec ...` — rather than invoking bare.
- Has real web search built in; reliable for pricing/fact lookups when prompted to search and cite.

## OpenCode
- Binary: `%USERPROFILE%\.npm-global\opencode.cmd`
- Invoke: `opencode run "<prompt>"`, or `opencode run --model mimo/mimo-v2.5-pro "<prompt>"` for the custom MiMo provider
- Config: `%USERPROFILE%\.config\opencode\opencode.json`
- Has direct file-read tools — genuinely inspects real files on disk when asked to (verified: correctly read this machine's actual Claude Code JSONL session files to reverse-engineer their schema, rather than guessing from general knowledge).

## Google Antigravity CLI (agy)
- Binary: `%USERPROFILE%\AppData\Local\agy\bin\agy.exe`
- Invoke: `agy --dangerously-skip-permissions -p "<prompt>"` (the skip-permissions flag is needed for non-interactive research tasks — without it, tool calls requiring permission get auto-denied in headless mode)
- **Do not** invoke `Antigravity.exe` directly (the IDE binary) — it launches a GUI window, not a CLI session. `agy` is the separate real CLI.
- Produces well-cited, decisive research reports; has been reliable this session for architecture/technology research (RAG system design, glassmorphism/DWM implementation).

## Claude Code itself (coordinator)
- Own memory: `%USERPROFILE%\.claude\projects\<project-folder>\memory\` — note there can be more than one project-folder for the same physical directory (junction-based aliasing), so check sibling memory directories if something expected isn't found.
- Own session transcripts: `%USERPROFILE%\.claude\projects\<project-folder>\<session-uuid>.jsonl` — the currently active session is the most recently modified `.jsonl` in the relevant project folder.

## GCP / infra tooling used alongside delegation
- `gcloud`: `%USERPROFILE%\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\gcloud.cmd` — from PowerShell, not Bash (Bash's invocation of the `.cmd` wrapper mangles paths containing spaces). `gcloud compute ssh` needs `--tunnel-through-iap` when direct SSH times out, and the remote command string works best as `--command="..."` (equals-sign form) rather than a bare `--command "..."` positional, which PowerShell's quoting can mangle.
