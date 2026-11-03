# codexengine: the Codex CLI as a shuttle engine, with its home and launch

Adds `internal/shuttleengine/codexengine`, a second shuttle engine beside `claudeengine` that drives the Codex CLI's interactive TUI in a reed pane.
It registers in `internal/shuttleengines` as `codex`, with its own config module `codex.yaml`.
No role runs on Codex yet; this lays the engine, its home and its launch for the later parts of the `codex-engine` project.

## What changes

- **Engine.** `codexengine` implements `shuttleengine.Engine`: launch, startup classification, send, idle and busy readings, completion, interrupt and context size.
  The first turn is the positional prompt pointer, and the pane's shell exits after Codex, so a Codex exit reads as a dead pane.
  Hooks given by `-c` write SessionStart, UserPromptSubmit, Stop and PermissionRequest payloads to the run's events file, and PostToolUse touches the activity stamp.
  A Stop is the turn end.
  An interrupt fires no Stop, so a newest `turn_aborted` in the rollout marks the turn interrupted.
- **Shared lym Codex home.** Every launch runs under one home, `codex.yaml`'s `home`, set as `CODEX_HOME`; the operator's own Codex home is never written.
  The lym home links `auth.json` and `sessions` to the operator home's, so one login serves both and rollouts land where the operator's tools read them.
  lym keeps the trusted roots in its own record, `lym-trust.json`, and renders `config.toml` whole from it on each launch under a file lock.
  Prepare trusts the worktree root and a linked worktree's main checkout root, and drops recorded roots whose folder is gone.
- **Launch line.** `-a never -s workspace-write`, so no approval prompt shows.
  The records checkout is added as a writable root, so the session writes under the anchor's `.lym` as a Claude seat does.
  `configengine.Dirs` gains `Records`, filled by `fabricengine.ConfigDirs` with the weft checkout when it exists; no config layer reads it.
  Outside a hub the engine is told none and adds none.
- **Model aliases.** `sol`, `astra` and `luna` resolve to the codex engine in the built-in registry and in the `models.yaml` template, at `effort: medium`.
  The built-ins carry that default so a Codex spec naming no effort never launches at the provider's own default.
  A hub whose `models.yaml` predates the aliases still resolves them.
  `contracts/specs/llm-model-spec.md` states this.
- **File link.** `fslink.CreateFileLink` creates a file symlink on Linux and refuses on Windows before touching the filesystem.
  `PATTERN-fs-links.md` and the package doc are updated.
- **Startup check.** An optional `shuttleengine.StartupChecker` capability runs once a launch is past its startup gates.
  Codex writes its own `config.toml` and ignores lym's lock, so the engine reads the file back and warns about a trusted root that was dropped.
- **Env scrub.** `CODEX_*` variables join the engines' declared scrub, and `tmuxkit.ServerEnviron` strips them from a test server's environment.

## Design decisions

- The writable root is the weft checkout, the target of the `.lym` and `_lym` links, symlink-resolved as the trusted roots are.
  When it cannot be resolved it is kept as told, with a Warn.
  This replaces a writable root on the assistant report path.
- Trust is a lym-owned record rendered into `config.toml` rather than a `-c` flag, because Codex keys folder trust on a git root and a `-c` flag cannot set it.
- A committed `.codex/rules` folder in the worktree root or the anchor refuses the launch, since an allow rule there lifts the sandbox.
  A committed `.codex/config.toml` is named in a Warn, since its hooks run unreviewed.
- A login made inside the lym home leaves a regular `auth.json`; Prepare names it in a Warn and keeps it.
- Codex keeps no session registry, so liveness reads unproven, and the resume line starts no Codex because Codex mints its session id at the first turn.
- A send pastes the text and leaves the Enter to shuttle's guarded send, since Codex drops an Enter typed inside a burst of characters.
- Out of this change: skill loading, shell guards, fork audit, transcript reading and creel hooks on Codex.
  The engine refuses a spec that needs the publish guard.

## To check by hand

- Run a real Codex session in a hub and confirm the sandbox lets a write through the `.lym` and `_lym` links into the weft checkout.
  The build proves the root reaches the launch line; the write through the links against a live Codex was not run here.
- Run one real Codex launch against the pane readings (`›` composer, `Working (… esc to interrupt)` busy line, folder-trust prompt) and the hook events file; the suites use recorded fixtures.
- Confirm on a machine with an existing `models.yaml` that `sol`, `astra` and `luna` resolve without re-seeding.
- Confirm a Windows build refuses `CreateFileLink` and so refuses a Codex launch with a message naming the way forward.
