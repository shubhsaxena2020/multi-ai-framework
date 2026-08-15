# TOOLS_AND_MCP

**Platform note:** this file documents one real Windows setup (paths, `.cmd` shims, PowerShell quoting quirks) as a concrete worked example, not a universal spec. On Linux/macOS the binaries, shell (`ps aux` in `FAILURE_AND_RECOVERY.md` is Unix-native and will work there as-is), and path conventions will differ — replace the specifics below with your own tools' real invocation details; the pattern (bare-command-first resolution order, documenting known hangs, real config locations) is what's portable.

Real, verified binary locations and quirks for each CLI, as of 2026-07-29. `.npm-global` and `.local\bin` were added to the persistent User PATH on this date, so bare commands (`opencode`, `codex`, `claude`, `graphify`) should resolve in *new* shells — but any Claude Code session/terminal opened before that fix still needs the full path until restarted.

**Resolution order:** try the bare command first (cheap, and confirms whether the PATH fix has propagated to this shell). If it fails with "not found", fall back to the full path listed below. If a path below has gone stale (e.g. Codex's hash-suffixed folder changed after an update), glob for the binary rather than giving up — e.g. `Get-ChildItem -Path "$env:LOCALAPPDATA\OpenAI\Codex\bin" -Filter codex.exe -Recurse`. Don't default straight to the full path on every call; that's guaranteed to rot as installs update, and bare-first costs nothing when it works.

## Bypass-permissions flags — use by default on every delegated call (all 4 confirmed working, 2026-07-31)

All four CLIs have a real flag that stops them from pausing on interactive prompts (tool-permission confirmations, or in Codex's case, an onboarding-style clarifying question from an invoked skill) during unattended/background delegation. Real, concrete failures this caused before the fix: Codex stalled mid-run asking "How long should I spend?" from its own `firecrawl-deep-research` skill and exited without doing the research; OpenCode hit an `external_directory` permission request, got auto-rejected, and produced a truncated non-answer. Verified fix, each tested live with a real command:

- **Codex**: add `--dangerously-bypass-approvals-and-sandbox` to the `exec` invocation.
- **OpenCode**: add `--auto` to the `run` invocation ("auto-approve permissions that are not explicitly denied").
- **Hermes**: add `--yolo` ("bypass all dangerous command approval prompts").
- **Antigravity**: already used `--dangerously-skip-permissions` (unchanged, still correct).

This does not remove the need to also pre-answer skill-driven onboarding questions in the prompt itself (e.g. Codex's research-depth question) — the bypass flag stops *tool-permission* prompts, not a skill explicitly asking the model something in its own text. Keep doing both: bypass flag on the invocation, and depth/scope pre-answered in the prompt.

## Hermes Agent
- Binary: `%USERPROFILE%\AppData\Local\hermes\hermes-agent\venv\Scripts\hermes.exe`
- Invoke: `hermes -z "<prompt>" --yolo`
- Config: `%LOCALAPPDATA%\hermes\config.yaml` (custom_providers, model.provider must be the literal provider name — `model.provider: auto` silently skips custom provider resolution entirely)
- **Known failure mode:** has hung indefinitely with zero output on at least two real occasions (30+ min, no error, no output file content). Per `FAILURE_AND_RECOVERY.md`'s timing guidance, kill it (`kill -9 <pid>`) and retry, or switch to Codex/OpenCode, once it's clearly past the reasonable window for the task's own nature and the process check shows it isn't making progress. Separately, it also tends to buffer all output until the very end rather than streaming — zero interim output for 10+ minutes is not itself a sign of a hang; confirm via a non-blocking task-status check before assuming it's stuck.

## OpenAI Codex CLI
- Binary: `%USERPROFILE%\AppData\Local\OpenAI\Codex\bin\<hash>\codex.exe` — the hash folder changes on update; glob for `codex.exe` under `AppData\Local\OpenAI\Codex\bin\` if the known path stops working. Also available via the npm shim at `%USERPROFILE%\.npm-global\codex.cmd` once that's on PATH.
- Invoke: `codex exec --skip-git-repo-check --dangerously-bypass-approvals-and-sandbox "<prompt>"`
- Config: `%USERPROFILE%\.codex\config.toml` — one real example had `model = "gpt-5.6-sol"` set on 2026-07-29. **Don't copy that model string as-is** — model availability changes; whatever you set, verify the model string is still valid by testing with `-m <model>` before committing it to config, don't just guess a name.
- **Known failure mode 1:** occasionally hangs on "Reading additional input from stdin..." and never proceeds. Fix: pipe empty stdin explicitly — `echo "" | codex exec ...` — rather than invoking bare.
- **Known failure mode 2:** if the prompt's task resembles report-scale research, Codex may invoke its own `firecrawl-deep-research` skill, which asks an onboarding question ("How long should I spend: a few minutes, ~10-15 minutes, or exhaustive?") and — in a non-interactive `exec` call — just exits after asking, producing zero actual research. Fix: explicitly pre-answer this in the prompt itself, e.g. append "Run this at the Thorough depth tier (~10-15 minutes, 5-10 search queries, 15-25 sources) without asking any clarifying questions — proceed directly and produce the final report." The `--dangerously-bypass-approvals-and-sandbox` flag does NOT fix this failure mode (it's a skill-level text question, not a tool-permission prompt) — both fixes are needed together.
- Has real web search built in; reliable for pricing/fact lookups when prompted to search and cite.

## OpenCode
- Binary: `%USERPROFILE%\.npm-global\opencode.cmd`
- Invoke: `opencode run --auto "<prompt>"`, or `opencode run --auto --model mimo/mimo-v2.5-pro "<prompt>"` for the custom MiMo provider
- Config: `%USERPROFILE%\.config\opencode\opencode.json`
- Has direct file-read tools — genuinely inspects real files on disk when asked to (verified: correctly read this machine's actual Claude Code JSONL session files to reverse-engineer their schema, rather than guessing from general knowledge).
- **Known failure mode:** if a prompt implies local repo access it doesn't actually have in this delegation's working directory, it wastes its run attempting `Glob`/`Read` calls that fail, then may stop after just exploring without producing the requested final report — even with `--auto` set and exit code 0. Fix: explicitly tell it in the prompt that it does not have filesystem access to the relevant repos and should work entirely from the prompt's own context plus live web research, and instruct it to produce the actual final written output as its last message rather than stopping after exploration.

## Google Antigravity CLI (agy)
- Binary: `%USERPROFILE%\AppData\Local\agy\bin\agy.exe`
- Invoke: `agy --dangerously-skip-permissions --mode accept-edits -p "<prompt>"` (the skip-permissions flag is needed for non-interactive research tasks — without it, tool calls requiring permission get auto-denied in headless mode; `--mode accept-edits` is separately required — see `FAILURE_AND_RECOVERY.md`, its default mode can fabricate a "done" report with zero real changes)
- **Timeout for large tasks:** the `--print-timeout` flag defaults to 5m. For tasks involving multi-file reads/writes, 3+ web searches, or large context, use `--print-timeout 10m0s` (or `15m0s` for very large tasks). Quick fact-checks are fine with the default.
- **Do not** invoke `Antigravity.exe` directly (the IDE binary) — it launches a GUI window, not a CLI session. `agy` is the separate real CLI.
- Produces well-cited, decisive research reports; has been reliable this session for architecture/technology research (RAG system design, glassmorphism/DWM implementation) and cross-verification passes (independently reproduced another delegate's platform-identification findings, confirmed accurate on WebFetch spot-check).

## Qwen Code (added 2026-08-15)
- Binary: `%USERPROFILE%\AppData\Roaming\npm\qwen.cmd` (npm global install: `npm install -g @qwen-code/qwen-code@latest`, requires Node.js 22+)
- Invoke: `qwen -p "<prompt>"`
- Config: `%USERPROFILE%\.qwen\settings.json` — `modelProviders.<authType>` is an array; entries beyond the first function as a **real, native ordered fallback chain** (confirmed via official changelog, v0.19.6+, July 2026) — triggers on 429/503/529 after each entry's own `generationConfig.maxRetries` is exhausted, resets to primary every new turn. Auth/quota/config errors are NOT covered, fail immediately. This is the only one of the four newly-added CLIs with a genuine native fallback mechanism; the others need a wrapper script (see `llm-fallback-chains` companion repo).
- **Real bug caught in this machine's own setup, worth watching for elsewhere**: entering an API key via `/auth` does not validate it against the URL you also enter — it's possible to save a syntactically-fine but functionally-wrong pairing (e.g. an NVIDIA-formatted key against a `baseUrl` for a completely different provider) with zero error at setup time; it only surfaces as an auth failure on first real use. Always test with a real `-p` call after setup, not just trust that `/auth` completing without error means it's correctly wired.
- Skills live at `.qwen/skills/` (same `SKILL.md` format Claude Code/Codex/OpenCode use — directly copy-compatible). MCP servers go under `mcpServers` in the same `settings.json`, standard `{command, args, env}` shape.

## Kiro CLI (added 2026-08-15)
- Binary: `%LOCALAPPDATA%\Kiro-Cli\kiro-cli.exe` — install via PowerShell `irm https://cli.kiro.dev/install.ps1 | iex` (the `curl ... | bash` install command in Kiro's own docs is Linux/macOS only and will silently fail under Git Bash on Windows; use the `.ps1` one). **The installer's own success message claims it installs to `C:\Program Files\Kiro-Cli\` — that's wrong, it actually lands in `%LOCALAPPDATA%\Kiro-Cli\`.** Check the real location directly rather than trusting the printed path.
- Invoke: `kiro-cli chat --no-interactive "<prompt>"`
- **No native fallback chain, and no wrapper-script workaround is possible either**: unlike the other three tools here, Kiro CLI has no custom OpenAI-compatible endpoint / BYOK option — `model` only selects from Kiro's own hosted catalog (Bedrock-routed), billed against Kiro's own subscription. There's no config surface to point it at NVIDIA/OpenRouter/anything else. See `NOT-POSSIBLE.md` in the `llm-fallback-chains` companion repo.
- **Real constraint, verify before relying on it for automated delegation**: headless/scriptable use (`--no-interactive`, the mode delegation needs) requires a paid plan for `KIRO_API_KEY` generation ($20+/mo Pro tier). The free tier (50 credits/month) is browser-OAuth-interactive only — it does still work for `--no-interactive` once logged in via browser once, but the credit ceiling makes it unsuitable as a primary automated delegate; treat it as an occasional/light-use tool, not a default choice.
- Skills at `.kiro/skills/` (same SKILL.md format). MCP servers: create `.kiro/mcp.json` with a standard `mcpServers` block — its own agent-config schema has an `includeMcpJson: true` default flag confirming this is the expected global MCP config location.

## Goose (added 2026-08-15)
- Binary: `%USERPROFILE%\goose\goose.exe` — install via Git Bash: `curl -fsSL https://github.com/aaif-goose/goose/releases/download/stable/download_cli.sh | bash`. Runs natively on Windows (no WSL needed).
- **Known failure mode**: the installer's own `goose configure` auto-launch at the end of install fails in any non-interactive shell (`bash: line 362: /dev/tty: No such device or address`) — the binary itself installs fine despite this error, only the interactive setup wizard fails. Configure via environment variables instead (see below), don't treat the install as failed.
- Config: no native provider config file needed — set `GOOSE_PROVIDER`, `GOOSE_MODEL`, and the matching `<PROVIDER>_API_KEY` env var (e.g. `OPENROUTER_API_KEY`) directly; env vars take precedence over `config.yaml` if both are set. Real confirmed-working combo: `GOOSE_PROVIDER=openrouter`, `GOOSE_MODEL=nvidia/nemotron-3-ultra-550b-a55b:free`. **A different free-tier model slug that looks equally valid may not actually be free right now** — OpenRouter's free-tier roster changes; a model returning a real, live, currently-served model list on a 404 is the fastest way to check, don't assume a model name from documentation/memory is still free.
- **No native fallback chain** (confirmed: single active `GOOSE_PROVIDER`/`GOOSE_MODEL` only, intra-provider retry/backoff exists but no cross-provider failover) — needs a wrapper script; see `llm-fallback-chains` companion repo.
- Since persistent Windows user-env-var writes can get blocked by the harness's own auto-mode classifier (a legitimate gate on system-level state changes), the practical fix is a `.cmd`/`.ps1` wrapper script in a PATH-visible `bin/` folder that sets the env vars per-invocation rather than trying to persist them to the registry.
- MCP servers: `extensions:` block in `%USERPROFILE%\.config\goose\config.yaml`. Real, verified field names (confirmed against actual docs, not the first search result which had it slightly wrong): `type: stdio`, `name`, `enabled: true`, `cmd` (not `command`), `args`, `envs` (not `env`), `timeout`.

## Factory Droid (added 2026-08-15, currently non-functional)
- Binary: `%USERPROFILE%\bin\droid.exe` — install via PowerShell `irm https://app.factory.ai/cli/windows | iex`.
- Invoke: `droid exec "<prompt>"`
- **Confirmed dead without a paid plan, no BYOK workaround exists**: browser OAuth login succeeds and a real auth keyring gets created, but every single call — even a trivial one-word test — fails with `HTTP 402: No active subscription found. Subscribe to start using Droid.` This contradicts one piece of earlier delegate-sourced research that suggested a BYOK (bring-your-own-key) option existed for OpenAI/Anthropic accounts — direct testing (checking `droid exec --help` and `droid --help` for any API-key/model-override flag) found no such option in the actual CLI. Treat that kind of "there's probably a workaround" claim as unverified until tested directly, same standard as any other delegate claim.
- Not worth adding to a delegate rotation unless/until a paid Factory subscription is purchased.

## Claude Code itself (coordinator)
- Own memory: `%USERPROFILE%\.claude\projects\<project-folder>\memory\` — note there can be more than one project-folder for the same physical directory (junction-based aliasing), so check sibling memory directories if something expected isn't found.
- Own session transcripts: `%USERPROFILE%\.claude\projects\<project-folder>\<session-uuid>.jsonl` — the currently active session is the most recently modified `.jsonl` in the relevant project folder.

## GCP / infra tooling used alongside delegation
- `gcloud`: `%USERPROFILE%\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin\gcloud.cmd` — from PowerShell, not Bash (Bash's invocation of the `.cmd` wrapper mangles paths containing spaces). `gcloud compute ssh` needs `--tunnel-through-iap` when direct SSH times out, and the remote command string works best as `--command="..."` (equals-sign form) rather than a bare `--command "..."` positional, which PowerShell's quoting can mangle.
