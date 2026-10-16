# Decision record — lyx shuttle open chooses the permission mode

## Goal

`lyx shuttle open` takes a `--permission-mode` flag, so an operator session opens in bypass under its lyx strand name for the role (`<shortname>[:<slug>]:<role>`, reed-numbered) instead of being removed and relaunched through `lyx reed add` under a generic name.

## Scope

In:

- A `--permission-mode` flag on `lyx shuttle open`, values `bypass` and `prompt`, empty for the engine's interactive default.
- A `PermissionMode` field on `shuttleengine.OpenSpec`, carried to the engine's `OpenLaunch`.
- `claudeengine`'s open launch realizing the mode on its fresh line and on its resume line, the fallback fresh line included.
- The permission mode on `Runner.Open`'s existing "opened session" log line.
- The docs the change touches: the `open` command's help text, the `OpenSpec` and open-launch paragraphs of the `shuttleengine` and `claudeengine` package docs, and the shuttle row of `docs/overview.md`.

Out:

- A config default for the open permission mode, in `shuttle.yaml`, `claude.yaml` or anywhere else.
- Any change to `lyx shuttle run`'s permission handling, to `validatePermissionMode`'s rules, or to the orch's own `permission_mode`.
- Shell guards or `--settings` for an opened session; it keeps taking the worktree's own settings.

## Decisions

### D1 — A flag, no config default

`lyx shuttle open` gains `--permission-mode <mode>`.
Empty, the flag's default, keeps today's behaviour: the session prompts.
Rationale: an empty default changes no existing caller, and the entry's "a flag or a config default" is met by the flag alone; a config key adds a resolution layer nobody asked for (YAGNI).

### D2 — Permission mode is engine vocabulary on `OpenSpec`

`OpenSpec` gains `PermissionMode string`, documented as caller-owned engine vocabulary exactly as `Spec.PermissionMode` is.
The CLI and `Runner.Open` pass it through uninspected; the engine is the sole validator and realizer, and an engine implementing `SessionOpener` honours the mode or refuses it.
The flag's help text names the mode words `bypass` and `prompt` and never a provider's flag.
Rationale: `Spec.PermissionMode` already sets this split (`shuttleengine/doc.go`), and `PATTERN-shuttle-provider-seam` keeps provider specifics, such as what `bypass` maps to, under the engine's package.

### D3 — Claude validates with the existing interactive rule

`Claude.OpenLaunch` resolves the mode through `validatePermissionMode(mode, true)` before it mints a session id, alongside its effort and model checks.
`bypass` adds `--dangerously-skip-permissions`, `prompt` and empty add nothing, and any other value is refused with the existing error naming the legal values.
`Runner.Open`'s existing refusals that run before `OpenLaunch` (unknown engine, covering pause, engine with no `SessionOpener`) keep reporting ahead of a mode refusal.
Rationale: an opened session is always interactive, so the existing interactive branch of the rule is the open rule; a second validator would drift from it.

### D4 — The resume line carries the mode too

`buildOpenCmd` takes the resolved skip flag and adds `--dangerously-skip-permissions` to both the fresh and the resume line, so `ResumeCmd`, which is the resume line or-else the fresh line, keeps bypass on both branches.
Rationale: reed replays `ResumeCmd` verbatim on a resume, and a resume line without the flag would bring a bypass session back prompting, the downgrade `buildResumeCmd`'s comment records for runs.
The line stays one pipeline with no environment assignment, so `OrElse` still joins it.

### D5 — Bound of the bypass edge

What it lets through: an opened session in bypass runs every tool without a permission dialog.
What bounds it:

- It is opt-in per call; nothing but an explicit `--permission-mode bypass` turns it on.
- The session still takes the worktree's own settings, and Claude Code applies their deny rules and hooks in bypass mode too (Claude Code behaviour, not lyx code).
- It is one interactive session per call in the caller's own worktree, in its own window an operator watches, refused under a covering pause, unchanged from today.
- It carries none of shuttle's run guards (no `--settings` from shuttle), as an opened session does today in prompt mode; bypass removes only the dialog that stood in for them.
- The opened-session log line records the mode, so a bypass open is traceable afterwards (`PATTERN-spawn-observability`).

## Constraints

- `PATTERN-shuttle-provider-seam`: the `bypass`→flag mapping lives only in `internal/shuttleengine/claudeengine`; `shuttleengine` and `shuttlecli` carry the string blind.
- `PATTERN-cli-cobra`: the flag is a cobra flag on the existing `open` subcommand.
- `PATTERN-shell-mechanics-seam`: the launch line keeps being built through `internal/shell`.
- Docs land in the same commit (CLAUDE.md).

## Auto-mode assumptions

- A1: The flag's values mirror `Spec.PermissionMode`'s vocabulary, `bypass` and `prompt`, so one vocabulary serves `run` callers and `open` callers. (D2, D3)

## Open risks

- R1: The `ly:cli` index (`lyx help index`) lists commands, not flags, so the orch finds the flag only through `lyx shuttle open --help`; until it reads that it may still fall back to `lyx reed add` for a bypass session.

## Acceptance criteria

- AC1: `lyx shuttle open --role r --permission-mode bypass` opens a session whose fresh launch line carries `--dangerously-skip-permissions`.
- AC2: The same open's `ResumeCmd` carries `--dangerously-skip-permissions` on both the resume branch and the fresh fallback branch.
- AC3: An open with no `--permission-mode`, or with `prompt`, produces launch and resume lines without `--dangerously-skip-permissions`, identical to today's.
- AC4: An open with an unknown mode (for example `Bypass` or `auto`) is refused with an error naming the legal values, before a session id is minted, a strand is added or the marker directory is created.
- AC5: The CLI hands the flag's value to `Runner.Open` unchanged in `OpenSpec.PermissionMode`.
- AC6: The `open` help text describes the flag and its example uses `--permission-mode bypass`; the `shuttleengine` and `claudeengine` package docs and `docs/overview.md`'s shuttle row describe the flag, and no doc still says the open line passes no permission flag.

## Notes for the plan writer

- `buildOpenCmd`'s comment says the line passes "no permission flag"; that sentence changes with D4.
- `TestOpenLaunch` is a table over `OpenSpec`; AC1–AC4 fit as rows there. AC5 fits beside the existing `open` rows in `internal/shuttlecli/cli_test.go`.
