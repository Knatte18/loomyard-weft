# Guard steers name the allowed route for Python and go compile checks

Agents kept reaching for forms lym's PreToolUse guards refuse, a Python call or a raw `go build ./...` or `go vet ./...`, and retried in the allowed form a turn later at full context.
The refusal text only said what was refused, so the agent had to guess the allowed form.
This change makes the python and raw go rows answer with the way forward.

## What changes

In `internal/shellguard/rows.go`:

- The python row's notice and steer state that `python` and `python3` are refused in every lym session, a version check included, so agents stop probing for Python first.
  The steer also names jq for JSON and grep or awk for text, ahead of the existing edit-tool rule.
- The raw go row's steer and notice name a package-scoped `go build ./internal/x` or `go vet ./internal/x` as the compile check, never `./...` or `all`, before the gate rule.
  Each form is offered only while no `refuse_raw` prefix refuses it (`compileCheckSentence`, `refusedRaw`), so a steer never offers a command the same guard would deny.
  When the prefixes refuse every form, the sentence is omitted.

The `settings-golden.json` fixture in `internal/shuttleengine/claudeengine/testdata` is regenerated for the new texts.
`contracts/specs/loom-plan-spec.md` no longer spells the compile criterion as `go build ./...`, and `internal/shellguard/doc.go`, the `claudeengine` package doc and `docs/overview.md` describe the new steers.

## Design decisions

- Python stays refused; the guard still follows the shuttle config's `python` key.
- Only the guard texts change, with no new rows and no change to what is denied.
- A prefix longer than the compile form (for example `go build ./internal/x extra`) does not refuse it, since the prefix has to open the form's words.

## To check by hand

- Whether the new steer wording holds: compare how often forks of a later tapestry run get a Python or raw go refusal against the baseline of 6 refused turns across the five heaviest forks of the three latest runs.
- The trial of the same rule as a line in the rendered `CLAUDE.local.md` through heddle, named in the board entry, was not done here; it belongs to the same round as the `batching-in-claude-local` note.
- The new `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` and the added `raw-go` rows in `rows_integration_test.go` were not run by me; run them through `lym gate test ./internal/shellguard`.
