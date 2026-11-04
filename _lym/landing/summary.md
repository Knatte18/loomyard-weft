# Fix the Codex trial faults in shuttle run, shuttle open and codexengine

The first Codex engine trial (`docs/research/codex-engine-trial-home-sandbox-2026-11-03.md`) found four faults.
This change fixes them before the next Codex trials run.

## 1. `lym shuttle run --model` takes a model spec

`run --model luna[medium]` used to pass the string to shuttle.yaml's engine as a provider model id.
Claude then stopped with `model_not_found`, and the run sat until its timeout.

- `run --model` now resolves through the same path as `open --model` (`shuttleCLI.resolveModelSpec` in `internal/shuttlecli/open.go`).
  It takes `<alias>[<effort>]` or `<engine>:<model-id>[<effort>]`, and an alias picks its engine from the model registry, so `luna` runs on codex and `sonnet` on claude.
  A malformed spec or an unknown alias is refused before any engine prepares.
- The `--effort` flag is gone, since the spec's bracket carries the effort.
- A model the engine rejects now fails the run at once.
  `SessionSignal` gains `ModelRejected`, and claudeengine sets it on a `StopFailure` whose `error` is `model_not_found`.
  `Run.retryAPIError` ends such a run `api-error` at once, whatever `APIErrorRetries` says, because no re-send changes the model.
  As before, a run whose output files all exist ends `done`, and a pause covering the run holds the end.
  The run envelope reports the error as `apiError`.

## 2. `lym shuttle open` starts a Codex session

The codex engine had no `SessionOpener`, so `open --model 'luna[medium]'` was refused even though `--list-models` lists the Codex aliases.
`Codex.OpenLaunch` now does the following:

- It validates the model choice and the permission mode (bypass only).
- It readies the lym Codex home as `Prepare` does, so the folder trust is rendered and a `.codex/rules` folder still refuses before anything is written.
- It returns the Codex command with no hooks and no first turn, so the session waits at its composer.

The opened session carries no shell guard, no creel hooks and no session id, and it writes nothing into the session directory.
Its resume line is empty: Codex mints its session id at the first turn, so a revive starts no Codex and `Runner.Open`'s chain ends the strand.

`Prepare` and `OpenLaunch` share `readyHome` and `validateChoice`.
The launch line's command part is now `codexCommand`, which leaves out `--dangerously-bypass-hook-trust` when there are no hooks and leaves out the positional prompt when there is no first turn; a run's line is unchanged.
The `open` help now says that an engine unable to open a session is refused before anything is created, and that a codex session has no shell guard and no creel hooks.

## 3. A test wrote the real `~/.lym/codex`

`TestPrepare_AppendsStandingPassages` told the engine an empty board dir.
That hid the fixture's local codex.yaml layer, so the config fell back to the template's `~/.lym/codex` and `~/.codex`, and every gate run added `/tmp` roots to the operator's trust record.
The test now writes the fixture's homes as the board's base layer, and it asserts that the launch readied the fixture's lym home.
The stale `/tmp` roots left in `~/.lym/codex` are not removed here, as the entry says: the next real launch drops roots whose folder is gone.

## 4. The `.codex` check

The check reads `.codex/rules` and `.codex/config.toml` on disk whether git tracks them or not, while the package doc said "committed".
Codex reads both either way, so the presence check is the one the safety argument wants.
The code is unchanged, and the package doc, the comments and the messages now say "in the worktree, tracked by git or not".
The refusal's way forward fits any launch: remove the directory from the worktree (in a commit when git tracks it) and launch again, or launch on an engine other than codex.
It no longer names a review round's assist model key.

## Design decisions

- `--effort` was dropped rather than kept as an override beside the bracket, so `run` and `open` take the same flag with the same meaning.
- The rejected-model end lives in the shared API-error watch, keyed on a provider-neutral signal field; the engine alone decides which provider error means a rejected model (PATTERN-shuttle-provider-seam).
- An opened Codex session's resume ends the strand instead of starting a fresh Codex, because Codex cannot name the session to resume.
  `codex resume --last` was not used: it would pick the newest session of the shared lym home, which may belong to another worktree.

## Limits

- Only claudeengine reports a rejected model.
  Codex has no turn-end-on-error hook in lym's events file, so a Codex model the provider rejects still waits out the run's deadline.
  A Codex alias the registry does not hold is refused before launch.
- The `model_not_found` value comes from the trial's observation of Claude's `StopFailure`.
