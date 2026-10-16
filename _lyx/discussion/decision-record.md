# Decision record — lyx shuttle open chooses the permission mode

## Goal

`lyx shuttle open` takes a `--permission-mode` flag, so an operator session opens in bypass under its lyx strand name for the role (`<shortname>[:<slug>]:<role>`, reed-numbered) instead of being removed and relaunched through `lyx reed add` under a generic name.

## Scope

In:

- A `--permission-mode` flag on `lyx shuttle open`, values `bypass` and `prompt`, empty for the engine's interactive default.
- A `PermissionMode` field on `shuttleengine.OpenSpec`, carried to the engine's `OpenLaunch`.
- `claudeengine`'s open launch realizing the mode on its fresh line and on its resume line, the fallback fresh line included.
- The permission mode on `Runner.Open`'s existing "opened session" log line.
- The docs the change touches: the `open` command's help text, the `buildOpenCmd` and `Claude.OpenLaunch` doc comments, the `OpenSpec` and open-launch paragraphs of the `shuttleengine` and `claudeengine` package docs, and the shuttle row of `docs/overview.md`.

Out:

- A config default for the open permission mode, in `shuttle.yaml`, `claude.yaml` or anywhere else.
- Any change to `lyx shuttle run`'s permission handling, to `validatePermissionMode`'s rules, or to the orch's own `permission_mode`.
- Shell guards or `--settings` for an opened session; it keeps taking the worktree's own settings.

## Decisions

### D1 — A flag, no config default

`lyx shuttle open` gains `--permission-mode <mode>`.
Empty, the flag's default, keeps today's behaviour: the line carries no permission flag, so the session starts in the default mode its Claude Code settings set, prompting when none sets one.
Rationale: an empty default changes no existing caller, and the entry's "a flag or a config default" is met by the flag alone; a config key adds a resolution layer nobody asked for (YAGNI).

### D2 — Permission mode is engine vocabulary on `OpenSpec`

`OpenSpec` gains `PermissionMode string`, documented as caller-owned engine vocabulary exactly as `Spec.PermissionMode` is.
The CLI and `Runner.Open` pass it through uninspected; the engine is the sole validator and realizer, and an engine implementing `SessionOpener` honours the mode or refuses it.
The flag's help text names the mode words `bypass` and `prompt` and never a provider's flag.
Rationale: `Spec.PermissionMode` already sets this split (`shuttleengine/doc.go`), and `PATTERN-shuttle-provider-seam` keeps provider specifics, such as what `bypass` maps to, under the engine's package.

### D3 — Claude validates with the existing interactive rule

`Claude.OpenLaunch` resolves the mode through `validatePermissionMode(mode, true)` before it mints a session id, alongside its effort and model checks.
`bypass` adds `--dangerously-skip-permissions`, `prompt` and empty add nothing, and any other value is refused with the existing error naming the legal values.
Because `prompt` adds nothing, it cannot override a default mode the Claude Code settings set.
`Runner.Open`'s existing refusals that run before `OpenLaunch` (unknown engine, covering pause, engine with no `SessionOpener`) keep reporting ahead of a mode refusal.
Rationale: an opened session is always interactive, so the existing interactive branch of the rule is the open rule; a second validator would drift from it.

### D4 — The resume line carries the mode too

`buildOpenCmd` takes the resolved skip flag and adds `--dangerously-skip-permissions` to both the fresh and the resume line, so `ResumeCmd`, which is the resume line or-else the fresh line, keeps bypass on both branches.
Rationale: reed replays `ResumeCmd` verbatim on a resume, and a resume line without the flag would bring a bypass session back prompting, the downgrade `buildResumeCmd`'s comment records for runs.
The line stays one pipeline with no environment assignment, so `OrElse` still joins it.

### D5 — Bound of the bypass edge

What it lets through: an opened session in bypass runs every tool without a permission dialog.
What bounds it:

- It is opt-in per call; lyx turns bypass on only through an explicit `--permission-mode bypass`.
  A settings layer's `defaultMode` can still set the mode outside lyx, as today.
- The session loads Claude Code's own settings layers, the worktree's included, and Claude Code applies their deny rules and hooks in bypass mode too (Claude Code behaviour, not lyx code); lyx installs no rule or hook of its own on an opened session.
- It is one interactive session per call in the caller's own worktree, refused under a covering pause, unchanged from today.
  It takes no turn until text reaches its pane through a channel that types there: the operator in its window, `lyx shuttle send <guid>`, `lyx creel inject`, and the reminder line a `lyx creel post` types into the idle receiver.
  Each channel types into a live reed strand with no run record, so a session driven through any of them runs every tool without a dialog and with no shuttle guard, bounded by that channel's own rules:
  - `send` refuses under a covering pause.
  - Creel lets EXTERNAL callers, programs and orch strands inject on any strand, and any other strand on its own strand only.
  - Any strand of the hub may post to the session, and so start a turn through the reminder.
  - Creel's sender identity boundary is the operator's user: any process of that user posts as EXTERNAL or as a program, and the 0700 lyxtemp root is the boundary.
  - A covering pause holds creel's injects and reminders too.
- It carries none of shuttle's run guards (no `--settings` from shuttle), as an opened session does today without the flag; bypass replaces whatever default mode the settings set with skipping every dialog.
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

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d5-bypass-driver-reach-omits-creel (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - card3-drops-open-refusal-envelope-coverage (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- AC1: `lyx shuttle open --role r --permission-mode bypass` opens a session whose fresh launch line carries `--dangerously-skip-permissions`.
- AC2: The same open's `ResumeCmd` carries `--dangerously-skip-permissions` on both the resume branch and the fresh fallback branch.
- AC3: An open with no `--permission-mode`, or with `prompt`, produces launch and resume lines without `--dangerously-skip-permissions`, identical to today's.
- AC4: An open with an unknown mode (for example `Bypass` or `auto`) is refused with an error naming the legal values, before a strand is added or the marker directory is created.
- AC5: The CLI hands the flag's value to `Runner.Open` unchanged in `OpenSpec.PermissionMode`.
- AC6: The `open` help text describes the flag and its example uses `--permission-mode bypass`; the `shuttleengine` and `claudeengine` package docs and `docs/overview.md`'s shuttle row describe the flag, and no doc still says the open line passes no permission flag.

## Notes for the plan writer

- `TestOpenLaunch` is a table over `OpenSpec`; AC1–AC4 fit as rows there. AC5 fits beside the existing `open` rows in `internal/shuttlecli/cli_test.go`.
