---
verdict: APPROVED
---

## Scope

The board entry (issue #702) asks that the stencilstore dev-build warning never print from creel verbs, either by printing once per refreshing process or by printing only from verbs that read stencils.
The decision record has no `Added after Discussion` entries, so the board body is the measure.

PR 713 at `3e4762451` makes three changes against `main`:

- `internal/creelcli/cli.go` puts `clihelp.SkipStencilSeedAnnotation` on the `creel` group and on every child of the group.
  The hidden `daemon` is one of those children, and no creel subcommand has children of its own, so the root pre-run's seed pass, which is the source of the warning, never runs for a creel command.
- `cmd/lyx/stencilseedgate_test.go` adds one row per creel command to `TestSkipStencilSeed_HonoursTheAnnotation`, with no new top-level test.
- `internal/creelcli/doc.go` states the skip.

This meets the entry's "never from creel verbs" outcome, and the change goes no further than that.
The refresh notice's own wording and once-per-pass behaviour stay unchanged, which the entry allows, since its expected outcome is an either-or.
The decision record has one open risk: a hub that runs only creel verbs never seeds its stencils.
That risk names no parent correction and needs no escalation.

## Overlap

- `creel-cli-output-fixes` (row 9, Plan-Burler-Review): it also changes `internal/creelcli`, at `runVerb`, `writeRefusal`, the verb bodies and `doc.go`.
  The overlap is text only.
  That run's work is the creel output path; this run's is the command annotations in `Command()` and one sentence in `doc.go`.
  The two designs do not conflict: the peer's decision record names this run, expects it to remove the warning line, and checks its own criteria against the command's own stderr writer.
  Landing order: this run (row 22) lands first, and `creel-cli-output-fixes` takes these changes at Merge-In.
- `gate-wait-cache-cap` (row 17) and `landing-trailer-text` (row 0, order unknown) change no file this run changes.
- No parent commits are listed.
