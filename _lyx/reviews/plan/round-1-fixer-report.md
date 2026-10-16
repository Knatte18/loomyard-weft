# Fixer report — plan, round 1

## Changes

- **F1 (MEDIUM, decision), fixed differently.**
  The review proposed deleting the clause "; a result outside the worktree root (`..` or `../…`) passes" from card 2's Intent.
  I did not only delete it, because the rule's "not a directory on disk under `worktreeRoot`" could then be implemented as a stat of the joined path, which finds a sibling of the worktree on disk.
  The Intent now says a result outside the root "is under no root and is no created directory, so it is missing without a stat".
  The reporting row "the overview chain `go test ../outside`, with an empty card" is added to `TestValidate_VerifyPackageMissing`'s rows that report.
  No target or verify command changed.
- **F2 (MEDIUM, design), fixed as proposed.**
  Card 2's rows that report nothing gain `true | cd internal && go test ./nope`, `false || cd internal && go test ./nope` and `go -C .. test ./nope`.
  A sentence after the list says each scan-end row, from `cd $DIR` to `go -C ..`, would report `./nope` if the scan went on.
- **F3 (LOW, consistency), fixed as proposed.**
  Card 4's read-error bullet now says that when the re-step is already used, the loop stops at once with `status-unreadable: <error>` and the read-error way forward, with no re-read.
  A new subtest of `TestLoop_InterruptedChildFillsTheErrorForm`, "a read error after a used re-step stops without a re-read", is added.
  Its first child fails through `failing` with a transient envelope, and `ReadStatus` delegates its first call to `readStatusAt` and then returns `errors.New("read lock unavailable")`.
  It asserts one child, the cause and way forward, two `ReadStatus` calls, and `loop.restep` naming the child's transient error.
  The card's twin doc bullets were updated too: refusal-spec's `step interrupted` row and `internal/shedverbs/doc.go`'s stop paragraph now name this stop.

- **Gate `edit-test-file` (three blocking findings), fixed.**
  Cards 1 and 2 named external test files by path under `**Edit:**`, which the overview's Shared Decision prescribed because, at this worktree's HEAD, a glyph on a `<dir>_test` unit derives the gate argument `./<dir>_test`.
  The installed `lyx` binary that runs this plan already carries that mapping, and its `edit-test-file` way forward names handles on `<dir>_test`, so the premise no longer holds for the binary running the plan.
  Card 1's Edit list now names `internal/planparser_test#TestCardTargetDirs` and `internal/batcher_test#TestPeakContext` in place of `internal/planparser/targetdirs_test.go` and `internal/batcher/estimate_test.go`.
  Card 2 drops `internal/planparser/validate_test.go` from Edit and declares `plan:internal/planparser_test#TestValidate_VerifyPackageMissing` -> `func TestValidate_VerifyPackageMissing(t *testing.T)` under Create, and its Tests paragraph now says the card declares the handle.
  The overview's Shared Decision is rewritten to "External-test members are named on their `_test` unit" with this rule.
  `lyx loom validate-plan` now returns `{"ok":true}`.

## Self-check

- consistency, LOW: card 2's Bound paragraph did not list the passes after a pipeline-stage `cd`, a `cd` after `||` and an unplaceable `-C`, which the new rows pin. Fixed as proposed: the Bound now reads "after a `cd` that ends the scan (unplaceable, in a pipeline stage or after `||`) or a `-C` that cannot be placed".
- consistency, LOW: card 4's new stop with no re-read reads against D5's "stops … only when the re-read fails too" (`_lyx/discussion/decision-record.md:90`). Fixed as proposed: the bullet adds "since D5's re-read runs under the existing single re-step and that re-step is spent".
- claims: no findings.
- way-forward: no findings.

## Disputed

None.

## Deferred

- D3's bound in `_lyx/discussion/decision-record.md:64` names only "after an unplaceable `cd`". The consistency fork raised it, but the file is outside this round's write surface. Card 2's Bound now states the full set.
- `_lyx/plan/lens-review.md:20` records a lens finding about test files named by path, rejected on the old Shared Decision's premise. It is left as written, because it records that past review, and the gate repair above supersedes it.
