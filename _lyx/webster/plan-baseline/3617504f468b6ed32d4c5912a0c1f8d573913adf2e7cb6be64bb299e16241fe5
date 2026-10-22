# Card 6 — reconcile-owner-outcome

**Edit:**
- `internal/fabricengine#OriginSeams`
- `internal/fabricengine#EntryOutcome`
- `internal/fabricengine#OriginReport.Summary`
- `internal/fabricengine#entryFacts`
- `internal/fabricengine#classifyEntry`
- `internal/fabricengine#classifyEntries` -> `func classifyEntries(scope ReconcileScope, recorded []string, branches []BranchOutcome, owners map[string]string) []EntryOutcome`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#originMover`
- `internal/fabricengine#originMover.move`
- `internal/fabricengine#EntryFactsForTest`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/fabricengine/doc.go`

**Create:**
- `internal/fabricengine#EntryOwnedElsewhere`
- `internal/fabricengine#OwnerReader`
- `internal/fabricengine#readOriginOwners`

**Uses:**
- `internal/fabricengine#ClassifyEntryForTest`

**Intent:**
D4's owner outcome, judged inside `ReconcileOrigin` (`PATTERN-origin-reconcile`, `PATTERN-owner-machine`).
`OwnerReader` is the owner question over one `board.json`: the effective owner of slug and whether it is another machine; the caller builds it over `boardengine` (card 7).
`OriginSeams` gains `OwnerOf OwnerReader`; a nil `OwnerOf` reads every entry as owned here.
`readOriginOwners` reads `board.json` at origin's fetched Bolt tip, the Bolt target's `OriginHead` (its objects a judge-only scope fetched by SHA, so it reads owners too and moves nothing), and asks `OwnerOf` once per slug of `slugs`; it returns the owner of each slug read as owned elsewhere, and nothing for a hub with no Bolt target or no origin head; a seam error is returned as the reconcile's error, an empty hostname's included, so no verdict is ever compared with an empty name (D1).
`ReconcileOrigin` calls it after judging and before the ended set, over the recorded pairs, and hands the map to the mover and to `classifyEntries`.
`EntryOwnedElsewhere` is the new entry state, `entryFacts` gains `OwnedElsewhere bool`, and `classifyEntry` keeps D4's precedence: ended, then owned_elsewhere, then stopped, then active.
`EntryOutcome` gains `Owner string`, set on an owned_elsewhere entry, whose `WayForward` is "`lyx adopt <slug>` from this machine's prime takes it over".
`OriginReport.Summary` gains one line per owned_elsewhere entry, "owned elsewhere: <slug> by <owner>; `lyx adopt <slug>` takes it over".
`originMover` gains the owners map, and `originMover.move` leaves an owned_elsewhere slug's `needs_replay` records branch where it is and reported, so only a fast-forward moves such an entry's branches and no commit made after the handover is replayed onto the new owner's tip by a reconcile; a live pair stays where it is as today.
Bound: the fast-forward-only rule holds in the reconcile only; the old machine's own weft push retry, `retryWeftPush`, still replays a rejected push onto origin's weft tip, since guarding it would need an owner read of origin's Bolt outside `ReconcileOrigin`, which the decision record's Constraints do not license.
`EntryFactsForTest` gains the `OwnedElsewhere` fact.
`doc.go`'s Origin reconcile section documents the seam, the state, its precedence and the fast-forward-only rule.
`TestReconcileOrigin_Judge`'s classifyEntry subtest gains rows: ended over owned_elsewhere, owned_elsewhere over stopping branches, and owned_elsewhere over a D3 stop.
`TestReconcileOrigin_Moves` gains rows over a hub whose origin Bolt names another owner through a fake `OwnerOf`: a run scope and a judge-only hub scope report the entry owned_elsewhere with the owner, its way forward and the summary line; a records branch both sides moved stays unmoved and reported `needs_replay`, while a fast-forward branch of the same entry moves; the owned_elsewhere entry's pair is never handed to the teardown seam, which is what `lyx cleanup` meets; an ended entry with an owner elsewhere reports ended; a nil `OwnerOf` reports the entry active; an `OwnerOf` error is the reconcile's error; a hub whose Bolt has no origin head asks `OwnerOf` nothing.

**ImpactSummary:** fabricengine's reconcile gains one seam, one state and one move rule; every existing caller passes a nil `OwnerOf` until card 7, keeping today's verdicts.

**Commit:** 6: fabric: the origin reconcile reports an entry owned elsewhere
