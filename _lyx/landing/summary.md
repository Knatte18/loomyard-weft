# Done-checks check glyph-less targets on disk and name quarry's search

Webster's post-batch done-checks (`planglyph.DoneChecks`) now check every target `lyx webster validate` accepts, and a failing check says enough to fix it.

## What changes

**Targets no glyph spells are checked on disk.**
Canonicalization turns a path-shaped card target into its self glyph through `glyph.Self`.
A path whose directory holds a space fails that call (`unit_bad_rune`), so the target stays a plain path, and quarry then rejected it before resolution (`no_separator`).
The batch failed `glyph-rejected` on every record and every recovery, even though the card had created the file (issues #805 and #809).
`DoneChecks` now sends a target that is still path-shaped to a disk check instead of the batched resolve.
A Create or Rename new side passes when the path exists, and a Delete or Rename old side passes when it does not.
The findings keep their check IDs, and the detail names the path checked.
The decision goes through a new shape-ledger gate, `planparser.IsDiskCheckedTarget`, so `planglyph` still makes no ref-shape decision of its own.

**A `not_found` Create names what quarry searched.**
When a Create target or Rename new side answers `not_found`, `DoneChecks` lists the worktree's declarations once through quarry's glyphs query.
The finding then names the files declaring the looked-up unit and every declaration elsewhere carrying the same name, or states that none exists (issue #823).
A test written in another package or under another name now reads differently from one the card never wrote.
The listing runs only when such a finding is due, so a clean batch pays nothing.

**A `glyph-rejected` done-check names the card amendment.**
Its way forward was `lyx webster recover-batch NN`, which re-runs the same check on the same plan and fails the same way.
It now says to amend the card's target, `lyx webster rebaseline --card NN`, then `recover-batch`.
A record holding uncheckable entries gets the fresh-restart steps, as the delete-referenced way forward already does.

## Design decisions

- `webster validate` does not refuse a spaced Create path, per the operator's decision on the board entry.
  The `loop-exited` halt reason is left to task `loop-exit-halt-names-its-cause`.
- The disk route covers Delete and Rename sides too, since a spaced path hits the same quarry rejection there.
- Quarry already resolves a `plan:` handle whose unit is a Go external test package (`sub_test#TestFoo`).
  A row in `TestDoneChecks_Verdicts` now covers it, so #823's misses came from where the tests were written, which the new detail exposes.
- The not-found detail lists every file of the unit rather than a truncated list, since the file the reader needs may be the one cut.
  A unit file that declares nothing does not appear, because the list comes from declarations.
- A listing error is wrapped in `ErrQuarryUnavailable`, so record-batch treats it as transient like a resolve failure, never as a verdict.
- `TestDoneChecks_RootFilenameCreateLanded` is folded into `TestDoneChecks_Verdicts`.
  Its second half pinned that a bare `LICENSE` target fails `glyph-rejected`, which the disk route now passes.
- `anyMemberMissing` and `searchedDescription` join the status tripwire's allowlist.
  Neither decides a verdict: the first only gates the listing, and the second reads `glyph.Glyph.Unit`, which the scan matches by name.

## Docs

`internal/planglyph/doc.go` and `internal/websterengine/doc.go` describe the disk route, the not-found detail and the new way forward.
`contracts/specs/loom-plan-spec.md` states that a target no glyph spells is checked on disk.
