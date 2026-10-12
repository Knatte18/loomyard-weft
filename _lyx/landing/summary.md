# Catch bad gate package paths, name loop status stops, document groups

Four intake bugs, one change each in `internal/planparser`, `internal/gatecli`, `internal/shedverbs` and the CLI-authoring docs.

## Gate steps no longer name an external test package as a directory

A card targeting `internal/x_test#TestY` produced the gate argument `./internal/x_test`, which is a quarry pseudo-unit and not a directory, so `lyx gate test` failed with `[setup failed]` on every run.

- `internal/planparser` maps an external-test glyph unit to its package directory in the ref-to-directory mapping behind `CardTargetDirs`.
  Webster's gate scope and `internal/batcher`'s read-set estimate both read it, and `internal/x` and `internal/x_test` merge into one entry.
  `vocabname.go`'s own `_test` trim now calls the same helper.
- A new blocking plan check, `verify-package-missing`, reads each card's `**Verify:**` value and the overview's `## verify:` chain.
  For `lyx gate test`, `go test`, `go build` and `go vet`, it reports a relative package argument whose directory neither exists nor is created by the plan up to that card.
  The finding names the card and the argument, and the corrected argument when it ends in `_test`.
  Only static spellings are checked; variables, `bash -c`, import paths, patterns and `.go` files pass.
  A Verify on or after a Move card is skipped, since a Move's destination lives in Intent prose.
- `lyx gate test` refuses a relative package argument that names no directory, before any slot path or prebuild.
  It exits 1 with a JSON error naming every such argument and, for an external test package, the corrected argument.
  `-C` arguments resolve against that directory, and import paths, absolute paths, patterns and `.go` files reach go test unchanged.
- `PATTERN-glyph-conversion-chokepoint` now names the one sanctioned adjustment, in `PATTERN.md` and in the background file.
  `contracts/specs/loom-plan-spec.md`, `contracts/specs/refusal-spec.md` and the package `doc.go` files are updated.

Closes #571
Closes #577
Closes #580
Closes #587

## The loop's status stops are diagnosable

The shed loop in `internal/shedverbs` stopped with `status-missing` for an absent and for an unreadable `status.json` alike, with no trace file and a generic way forward.

- An absent file keeps `status-missing`; a decode error stops with `status-unreadable: <error>`.
  A read-lock or raw read error is re-read once and stops with `status-unreadable` only when the re-read fails too.
- Both causes log a warning naming the status path, and the trace file handed to the interrupt hook is the loop's own log.
- The way forward is split by cause and names the status path and the loop log.
  `status-missing` names the recipe's reseed, a failed read names `lyx shed step <run> --until-stop`, and a decode error names re-stepping with a binary that decodes the file, or else reporting the stop to the run's parent.
- Decision for review: the status file is left as found at these stops, not written `failed`, since overwriting an absent or undecodable file would destroy the evidence.
  This departs from `PATTERN-failure-stop-visible` for that case only, and the pattern file now says so.
- `refusal-spec.md` changes the `step interrupted` row and adds a `loop status missing or undecodable` row.

Closes #581

## Runnable-group audience rule

A plan specified a runnable CLI group carrying an audience annotation, which `clihelp.Invocable` and `cmd/lyx/clitree_test.go` forbid.
No mechanical check can read a command's shape from card prose, so the rule is written where plans are written and reviewed.
`pattern/PATTERN-cli-cobra.md`'s audience bullet states it, `docs/overview.md` points at that bullet, and the Plan-Review rubric gains one check pointing at it.

Closes #588

## No change for the flaky sweep test

`internal/lyxtemp.TestSweep/two_concurrent_sweeps_both_succeed` does not reproduce on the bounded-retry code from `78e31aa06`.
On current code, `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass.

Closes #584

## To check by hand

- A flake that needs heavier machine load than those loops ran under may remain in `TestSweep`.
- `lyx gate test` now refuses a hand-written gate line with one wrong package argument, where it used to run the other packages.
- A real Go package directory whose own name ends in `_test` would map to its sibling directory; this repository has none.
- The #581 root cause is inferred from timing and the `routed_to` field, not from the lost run's files; the change holds whichever it was.
- In the shed loop's decode-error way forward, the re-step only helps after the file becomes decodable; check that the wording sends the operator to report the stop when it will not.
- A fork trimmed a `PATTERN-cli-cobra` line in `PATTERN.md` to stay within its size limit; check the trimmed wording.
