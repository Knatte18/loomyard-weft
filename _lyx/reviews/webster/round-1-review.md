---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/websterengine/guardstep.go:164
    summary: websterengine.cardCovers is a line-for-line copy of batcher.cardCovers, so the cover definition lives in two packages
    fix: Export the cover derivation once in internal/planparser, as CardCovers(card Card) []string with the doc comment of the current batcher copy. Delete cardCovers in internal/batcher/boundary.go and in internal/websterengine/guardstep.go, and call planparser.CardCovers from renameBoundaries and holdsFilesAfter. Then re-run lyx gate test on internal/planparser, internal/batcher and internal/websterengine.
---

### [LOW] The cover derivation is duplicated across batcher and websterengine

**Location:** internal/websterengine/guardstep.go:164, internal/batcher/boundary.go:62

**Issue:** Both packages define an unexported `cardCovers` with an identical body: the Delete refs plus the Old side of each Rename pair.
This is the set `planparser.RefCovers` takes as its `cover` argument, so a change to what counts as a cover (for example adding Move) must now be made twice or the batch boundary and the guard-step drop rule disagree.
The plan's Shared Decisions put every ref-shape decision in `planparser`; the cover set is the input to that decision, and two real call sites now need it.
The plan's cards do not forbid sharing it, so this is a convention departure, not a plan violation.

## Review notes

The rest of the range held up against the plan and the focus directive.

- Batching rule (cards 1 and 2): `renameBoundaries` decides coverage only through `planparser.RefCovers`, and `internal/batcher` tests no ref shape itself.
  The `boundaries[begin] < end` test sits inside the existing early break, which is sound because the inner loop only lowers `begin`.
  Both directions (a rename of an earlier target, a delete of one) have rows, and the unrelated-card rows still batch together.
  Only the cost batchifier's derivation changed, so a recorded `state.json` partition is untouched.
- Guard scan (cards 3 and 5): `impactset.FindGuardTests` and `impactset.MaxCommandLength` are the exports webster uses, with no second marker scan.
  Every walk error and every unopenable test file returns an error naming the path.
  `GuardScanError.File` is set at all three return sites.
- Stencils and gate (cards 4 and 6): the wording lives in the three fragments and the implementer body, and Go fills markers only.
  The scan runs before the first mutation in both `BeginBatch` and `recoverSpawn`, and the refusal text matches the plan and the new `refusal-spec.md` row.
  The guard step is placed before the two lints, an over-long step is dropped and reported, and a scan error becomes a prompt line.
- Per-card mechanical checks: the Rename group of card 3 leaves no `findGuardTests` or `maxCommandLength` anywhere in the tree, and the Create and Edit targets of cards 1, 2, 4, 5 and 6 are present.
  `lyx gate test` over `internal/planparser`, `internal/batcher`, `internal/impactset`, `internal/websterengine`, `contracts/stencils` and `cmd/lyx` passes, including the test budget.
- Test economy: the new tests each pin a behavior no existing test covered, and the two end-to-end rows reuse the shared fixtures rather than duplicating `TestScanGuardTests` or `TestCardGateCommand`.
  No redundancy or uncovered behavior to report.
