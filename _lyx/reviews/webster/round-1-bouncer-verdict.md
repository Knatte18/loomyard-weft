---
verdict: APPROVED
rationale: "Report APPROVED with F1-F3 (LOW/LOW/NIT), all fixed in c0d7f01dd, b89ef2c83 and bba52bb95; on HEAD bba52bb95 the card 20 Verify grep finds nothing, go build/vet/test pass, integration tests for fabriccli, fabricengine, cmd/lyx and tools/sandbox pass, and card 18's six moved files exist only under their new names with matching file comments."
---

# Webster-Bouncer round 1 verdict

Review range: `git diff $(git merge-base main HEAD)..HEAD`.
`product.parent` is `main`, read from `_lyx/shed/agent-naming/status.json`, and the merge base is `a5f0ce156`.

## The report

The round-1 report's own verdict is APPROVED.
It answers each focus entry with evidence.
It raises three findings, none of them blocking, and the Burler fixed all three in the same round.
The report has no `## Focus departures` section, so there is nothing to ratify or reject.

## The fixes, checked

- **F1, `c0d7f01dd`.**
  `clone`'s `Use` string, its mirror comment and the usage refusal in `clone.go` now start with `[--shortname <shortname>]`.
  The first help example passes `--shortname mono`.
  The `Long` text already said a fresh bind needs `--shortname`, so help and behaviour now agree.
- **F2, `b89ef2c83`.**
  Three docs now pass `--shortname` on a fresh-bind clone:
  README's Getting started, `docs/sandbox-hub.md` step 4 (`--shortname lyt`, the value `tools/sandbox/main.go` passes) and SANDBOX-FABRIC-SUITE F13.
  F13's "it always did" note is gone.
  The other `lyx fabric clone` examples still without the flag clone `--reset` or `--subpath` against wefts that are already bound, where the flag is optional, so they are correct.
- **F3, `bba52bb95`.**
  The fabric bullet in `docs/overview.md` is now one sentence per line, with breaks at semicolons and clause boundaries.
  Its continuation lines are indented, and no wording changed.

## What I checked myself on HEAD

- Card 20's Verify grep exits 1, which means no match, so the negated check passes.
- `go build ./...`, `go vet ./...`, `go test ./...` and `go vet -tags smoke ./...` pass.
- `go test -tags integration` passes over fabriccli, fabricengine, cmd/lyx and tools/sandbox.
- **Card 18's Move group.**
  No `code*` file remains in `fabricengine` or `fabriccli`, and each of the six new files opens its file comment with its own new name.
  `git show -M30%` pairs all six as renames.
  The report says the 37–45% similarity follows from the card's own one-commit shape, which this segment cannot change.
  I ratify that, and no package or import moved.
- `reedengine/strand.go`'s two refusals match cards 18 and 19 word for word.
  Both `refusal-spec.md` rows name `.lyx-shortname` and `lyx fabric shortname <shortname>`.

Nothing remains open.
