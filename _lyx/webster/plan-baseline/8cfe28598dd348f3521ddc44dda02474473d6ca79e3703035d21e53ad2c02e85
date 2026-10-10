# Card 11 — done-check-details-carry-the-lookup

**Edit:**
- `internal/planglyph#doneCheckVerdicts` -> `func doneCheckVerdicts(worktreeRoot string, entries []doneCheckEntry, index map[string]quarry.ResolveResult) ([]Finding, error)`
- `internal/planglyph#DoneChecks`
- `internal/planglyph/doc.go`
- `contracts/specs/refusal-spec.md`
- `internal/planglyph/donecheck_test.go`
- `internal/planglyph/status_completeness_test.go`

**Uses:**
- `internal/planglyph#classifyAmbiguity`
- `internal/planglyph#candidateList`
- `internal/planglyph#resolveKeyFor`

**Intent:** Make the done-check findings say what was looked up and what quarry answered (decision D5), and accept a partitioned member in the Create direction (decision D6).
In `doneCheckVerdicts`, `resolved` is true for `found`, `multipart`, and an `ambiguous` answer `classifyAmbiguity` reports partitioned; `stillExists` stays "anything but `not_found`", so the Delete direction and the Rename old side keep blocking on any surviving declaration.
Bound on the weakened guard: a Create or Rename new side lands over an ambiguous answer only when its candidates provably never coexist in one build environment, under the classifier's cap and its fallback to not partitioned on any read or parse failure, as card 10 states.
Every `create-not-done`, `delete-not-done` and `rename-not-done` detail names the resolve key that was looked up (`entry.key`, the glyph behind a `plan:` handle, beside the card's own spelling), quarry's status, for `not_found` the unit status (the unit exists and the member is missing, or the unit is missing, read from `ResolveResult.Unit`), and for `ambiguous` each candidate's file with its build constraint through `candidateList`.
`DoneChecks` passes its `worktreeRoot` through.
Update the `create-not-done, delete-not-done, rename-not-done` bullet of `doc.go` with the detail's contents and the partitioned rule.
In `refusal-spec.md`, the "card not done" row's Way forward says the detail carries the lookup: the key looked up, quarry's status, the unit status for a missing member, and each candidate's file and constraint for an ambiguous one.

Tests: extend `TestDoneCheckVerdicts_Rules` (`donecheck_test.go`) with a `wantDetail` column asserting the key, the status, the unit status for both `not_found` shapes, and the candidate files and constraints for an ambiguous answer; add fixture-backed rows resolving a member declared in `//go:build linux` and `//go:build !linux` files, which lands a Create and a Rename new side and still blocks a Delete (`delete-not-done`) and a Rename old side.
Update `runDoneCheckVerdict` in `status_completeness_test.go` for the new parameter; its synthetic ambiguous result has no candidate files, so it stays not partitioned and `expectedOutcomeByStatus` is unchanged.

**ImpactSummary:** One unexported re-sign with callers in donecheck.go and two test files; only the Create and Rename-new verdicts over a partitioned member change, and every done-check detail gains the lookup.
