# shuttle: run takes a model spec, open starts codex, test spares ~/.lym

Fixes the faults the first Codex engine trial found (`docs/research/codex-engine-trial-home-sandbox-2026-11-03.md`).

## What changes

- `lym shuttle run --model` takes the model spec `lym shuttle open` takes: `<alias>[<effort>]` or `<engine>:<model-id>[<effort>]`, such as `sonnet[medium]` or `luna[high]`.
  The registry picks the engine from the alias, so `luna` runs on codex whatever `shuttle.yaml` names.
  Before, the string went to `shuttle.yaml`'s engine as a provider model id, so `luna[medium]` reached Claude, which stopped with `model_not_found`.
  The `--effort` flag is removed; the effort is the spec's bracket.
  `run`'s output gains an `apiError` field when the outcome is `api-error`.
- A model the provider rejects ends the run at once with outcome `api-error`, whatever the spec's `APIErrorRetries`, since no re-send changes the model.
  The claude engine marks `model_not_found` as such through the new `SessionSignal.ModelRejected`; an engine that does not set it keeps the old behaviour, a wait to the timeout.
  Every `api-error` end now removes the run's own-window strand and saves its last pane capture to `died-capture.txt`, as a died run does, since the pane is still live and no caller is guaranteed to stop it.
- `lym shuttle open` starts a codex session through the new `codexengine.OpenLaunch`: Codex under the lym home with no prompt, no hooks and no shell guard, waiting at its composer.
  It readies the home and refuses under `.codex/rules` as `Prepare` does.
  Its resume line is empty, so a revive starts no Codex and the chain after it ends the strand.
  An engine without `SessionOpener` is still refused before anything is created; the `open` help says so.
- `TestPrepare_AppendsStandingPassages` keeps the fixture's codex home: the told board gets a base `codex.yaml` naming the fixture's homes, and the test asserts the trust roots land in the fixture's lym home.
  It no longer adds `/tmp` roots to the operator's `~/.lym/codex` and `~/.codex`.
- The `.codex` check now says what it does.
  The refusal on `.codex/rules` and the Warn on `.codex/config.toml` read the folders on disk, tracked by git or not, since Codex reads them either way; the package docs, comments and test names follow.
  The refusal's way forward is worded for the launch at hand: remove the directory from the worktree, in a commit when git tracks it, then launch again, or launch on an engine other than codex.

## Design decisions

- Both checks stay on-disk rather than tracked-only, because the safety argument is about what Codex loads, and an untracked allow rule lifts the sandbox as much as a committed one.
- `ModelRejected` is decided by the provider's engine and carried on the session signal, so the shuttle engine stays free of provider error strings.
- `run` and `open` share one `resolveModelSpec`, so the two verbs cannot drift on what a spec means.

## To check by hand

- The stale `/tmp` roots a past test run left in `~/.lym/codex` are not removed; the next real codex launch drops roots whose folder is gone.
- A rejected model has an at-once end only on the claude engine; this diff adds no codex signal that sets `ModelRejected`.
  Check that a codex run on an unknown model slug still ends in a bounded time.
- `lym shuttle open --model 'luna[medium]'` was exercised by unit tests only; open it once against a real Codex install to see the composer come up.
- Code review left `rejected-model-strand-left-live` open (design finding, severity medium): a rejected-model end must remove its own-window strand.
  Later fixes remove the strand on every `api-error` end in `endOnAPIError`; check that path in `internal/shuttleengine/apierrorretry.go` leaves no live pane.
