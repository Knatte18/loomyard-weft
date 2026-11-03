# codexengine: the Codex CLI shuttle engine, its home and its launch

Adds `internal/shuttleengine/codexengine`, a second shuttle engine that drives the Codex CLI's interactive TUI in a reed pane, and registers it as `codex` beside `claude`.
This is part 1 of project `codex-engine`: decisions D4, D5, D6, D7, D8 and D13 of the project's decision record, with the weft checkout as a Codex writable root (the parent's added decision).
The assistant seat, skills, shell guards and engine-keyed stencil blocks belong to the project's other parts.

## What changes

- **Registration and aliases (D4).**
  `internal/shuttleengines` registers `codex` with its config module `codex.yaml` (`binary`, `home`, `operator_home`, `efforts`, `skill_roots`) and its scrub, the `CODEX_` prefix.
  `tmuxkit.ServerEnviron` mirrors the new prefix, and the test that pins the two together covers it.
  The model-spec engine set gains `codex`, and both the built-in fallback and `models.yaml`'s template gain `sol`, `astra` and `luna` (`gpt-6.1-sol`, `gpt-6-astra`, `gpt-6-luna`) at effort `medium`.
  Prepare refuses an effort `codex.yaml`'s `efforts` does not list (`low`, `medium`, `high` by default), naming the key.
- **The shared lym Codex home (D5).**
  Every launch sets `CODEX_HOME` to `codex.yaml`'s `home` (`~/.lym/codex`); lym never writes the operator's `~/.codex`.
  Prepare links the home's `auth.json` and `sessions` to the operator home's.
  It keeps the trusted roots in `lym-trust.json` and renders `config.toml` whole from it under a file lock: the two `[tui]` settings, then one trusted project table per root.
  It trusts the worktree root and, for a linked worktree, the main checkout root read from the `.git` entries through `internal/dotgit`, and drops recorded roots whose folder is gone.
  A regular file, or a link elsewhere, in place of a home link is kept and named in a Warn with the way forward.
  A committed `.codex/config.toml` in the worktree, and the main checkout's when the worktree holds a `.codex/` folder, get a Warn.
  A committed `.codex/rules/` refuses the launch with a Warn.
- **The launch line (D6).**
  The line is the spike's: `--no-alt-screen -a never -s workspace-write`, the model and `model_reasoning_effort`, the `CLAUDE.md` fallback, the hooks and `--dangerously-bypass-hook-trust`.
  It adds `sandbox_workspace_write.writable_roots` naming the weft checkout, takes the prompt pointer as the positional first turn, and ends with `exit`, so a Codex exit leaves a dead pane.
  The resume line is `exit` alone.
  `prompt.md` opens with the standing passages heddle holds for engine `codex` and the spec's role.
- **Hook-fed events, completion and interrupt (D7).**
  `-c` hooks append a stamp line and the payload of `SessionStart`, `UserPromptSubmit`, `Stop` and `PermissionRequest` to the run's events file, and `PostToolUse` touches the activity stamp.
  `Stop` is the turn end and carries the reply; the session signals map the four hooks to a session start, a turn start, a turn end and an ask.
  A newest `turn_aborted` in the rollout the payloads name marks a turn interrupted.
  Liveness reads unproven, and the launch carries no creel hooks.
- **Pane driving (D8).**
  Startup classifies the spike's ready composer, the folder-trust prompt, and the hook-review prompt as still booting.
  The trust accept moves the caret onto `Trust and continue` and presses Enter, and presses nothing when that option is not next to the caret.
  The engine implements `InputBoxReader`, `InterruptGuard` (the `Working (… esc to interrupt)` line, Escape, a 2-second settle), `TextPaster` with `PasteSubmits` false and the `[Pasted Content N chars]` placeholder, and `SessionCycler`.
  The context size comes from the rollout's newest `token_count`.
- **A file link (D13).**
  `internal/fslink` gains `CreateFileLink`: a symlink on Linux, a refusal naming its way forward on Windows before anything is touched.
  `PATTERN-fs-links` names it.
- **A check after startup.**
  `shuttleengine` gains the optional `StartupChecker` capability, which the runner calls once a launch is past its startup gates.
  codexengine uses it to read `config.toml` back and warn about a trust root Codex's own write dropped.

## Design decisions

- **The weft writable root comes from the anchor's links.**
  Prepare is not told the weft checkout, so the engine takes the parent of the resolved target of the anchor's `.lym` and `_lym` links.
  That is the path Codex keys a linked path by.
  An anchor whose `.lym` and `_lym` are real directories, as outside a hub, gets no writable root, since they sit inside the workspace.
- **The write through the links is proven.**
  D6 required the build to prove it before D2 builds on it.
  `codex sandbox` (Codex 0.164.0, no model call) refused a write through `.lym` with `Read-only file system` without the root, and let writes through both `.lym` and `_lym` with the root.
- **Refusals before anything is written.**
  Prepare refuses a version pin, a permission mode other than bypass, a resumed session and the publish guard.
  It refuses the publish guard because no shell guard runs on Codex until the skills-and-guards part lands.
- **No session id.**
  Codex mints its own id at the first turn, so the launch names none.
  For the same reason the engine implements no `UsageReader`; the context reading goes through the Stop payload's rollout path instead.
- **Left for the other parts.**
  The prompt file does not yet open with D10's deny notice, and `skill_roots` is declared as D4 lists it but nothing reads it until skill loading lands.
  `config.toml`'s `model_availability_nux` table is rendered empty, since the twill spike recorded its name and not its content.
  Clear, compact and plugin reload type nothing, since none has been tried on Codex.

## Tests

Tier-0 tests cover the launch and resume lines, the hook settings and TOML quoting, Prepare's launch, prompt file and passages, and the home's links, record and `config.toml`.
They also cover concurrent launches into one home, each refusal leaving no artifact, each Warn, and the check after startup.
Parsing is pinned against the spike's hook payloads, rollout fixtures and pane captures.
A tier-1 test runs the recording and activity hooks under `sh` and reads what they write back through the parsers.
The registry, model-spec, config-registry and config-sync tests are updated for the new engine, aliases and module.
The one llm-tier smoke run the acceptance criteria name drives the assistant seat, which the project's last part builds.
