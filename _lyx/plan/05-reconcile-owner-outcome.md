# Card 5 — reconcile-owner-outcome

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
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#EntryFactsForTest`
- `internal/fabricengine_test#TestReconcileOrigin_Judge`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/fabricengine/doc.go`

**Create:**
- `plan:internal/fabricengine#EntryOwnedElsewhere` -> `const EntryOwnedElsewhere EntryState = "owned_elsewhere"`
- `plan:internal/fabricengine#OwnerReader` -> `type OwnerReader func(hubPath string, boardJSON []byte, slug string) (owner string, elsewhere bool, err error)`
- `plan:internal/fabricengine#InstallOwnerReader` -> `func InstallOwnerReader(r OwnerReader) (restore func())`
- `plan:internal/fabricengine#readOriginOwners` -> `func readOriginOwners(l *lyxcwd.Location, seams OriginSeams, targets []originTarget, outcomes []BranchOutcome, slugs []string) (map[string]string, error)`
- `plan:internal/fabricengine#ParentHoldsTaskWork` -> `func ParentHoldsTaskWork(l, pairLocation *lyxcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine_test#TestPushAnchored_OwnedElsewhereWeftIsNotReplayed` -> `func TestPushAnchored_OwnedElsewhereWeftIsNotReplayed(t *testing.T)`

**Uses:**
- `internal/fabricengine#parentHoldsTaskWork`
- `internal/fabricengine#ClassifyEntryForTest`

**Intent:**
D4's owner outcome, judged inside `ReconcileOrigin` (`PATTERN-origin-reconcile`, `PATTERN-owner-machine`).
`OwnerReader` is the owner question over one `board.json`: the effective owner of slug and whether it is another machine; the caller builds it over `boardengine` (card 6).
`OriginSeams` gains `OwnerOf OwnerReader`; a nil `OwnerOf` reads every entry as owned here.
`readOriginOwners` reads `board.json` at origin's fetched Bolt tip, the Bolt target's `OriginHead` (its objects a judge-only scope fetched by SHA, so it reads owners too and moves nothing), and asks `OwnerOf` once per slug of `slugs`; it returns the owner of each slug read as owned elsewhere, and nothing for a hub with no Bolt target or no origin head; a seam error is returned as the reconcile's error, an empty hostname's included, so no verdict is ever compared with an empty name (D1).
`ReconcileOrigin` calls it after judging and before the ended set, over the recorded pairs, and hands the map to the mover and to `classifyEntries`.
`EntryOwnedElsewhere` is the new entry state, `entryFacts` gains `OwnedElsewhere bool`, and `classifyEntry` keeps D4's precedence: ended, then owned_elsewhere, then stopped, then active.
`EntryOutcome` gains `Owner string`, set on an owned_elsewhere entry, whose `WayForward` is "`lyx adopt <slug>` from this machine's prime takes it over".
`OriginReport.Summary` gains one line per owned_elsewhere entry, "owned elsewhere: <slug> by <owner>; `lyx adopt <slug>` takes it over".
`originMover` gains the owners map, and `originMover.move` leaves an owned_elsewhere slug's `needs_replay` records branch where it is and reported, so only a fast-forward moves such an entry's branches and no commit made after the handover is replayed onto the new owner's tip by a reconcile; a live pair stays where it is as today.
The same replay recurs on the old machine's own push: `retryWeftPush`, after a rejected push of a pair's weft, fetches origin's Bolt as today and asks the process's installed `OwnerReader` (`InstallOwnerReader`, installed by the root pre-run in card 6, nil reading owned here) over origin's Bolt `board.json`; a pair slug owned elsewhere is not replayed, and the rejection stands as a failed push whose way forward names the owner and `lyx adopt <slug>`, so D3's and D6's bound holds: the old machine's pushes into origin's moved branches are refused and recorded, never replayed onto the adopter's tip.
`Fabric.Pull` is left as it is, since it replays onto the local branch only and every push from there meets the guarded retry.
`ParentHoldsTaskWork` exports `parentHoldsTaskWork` unchanged for Finalize's and InnerRun's carve-out (D6, D7), and `finalizeEntered` keeps calling the unexported one.
`EntryFactsForTest` gains the `OwnedElsewhere` fact.
`doc.go`'s Origin reconcile section documents the seam, the state, its precedence, the fast-forward-only rule and the guarded push retry.
`TestReconcileOrigin_Judge`'s classifyEntry subtest gains rows: ended over owned_elsewhere, owned_elsewhere over stopping branches, and owned_elsewhere over a D3 stop.
`TestReconcileOrigin_Moves` gains rows over a hub whose origin Bolt names another owner through a fake `OwnerOf`: a run scope and a judge-only hub scope report the entry owned_elsewhere with the owner, its way forward and the summary line; a records branch both sides moved stays unmoved and reported `needs_replay`, while a fast-forward branch of the same entry moves; the owned_elsewhere entry's pair is never handed to the teardown seam, which is what `lyx cleanup` meets; an ended entry with an owner elsewhere reports ended; a nil `OwnerOf` reports the entry active; an `OwnerOf` error is the reconcile's error; a hub whose Bolt has no origin head asks `OwnerOf` nothing.
`TestPushAnchored_OwnedElsewhereWeftIsNotReplayed` (integration) installs a fake `OwnerReader` reading the pair owned elsewhere, diverges the pair's weft from origin and pushes: the push returns the rejection with the owner and `lyx adopt` in its way forward, and origin's weft holds no local commit; with the reader reading owned here, the same push replays as today.

**ImpactSummary:** fabricengine's reconcile gains one seam, one state and one move rule, and its weft push retry one owner guard; every existing caller passes a nil `OwnerOf` until card 6, keeping today's verdicts.

**Commit:** 5: fabric: the origin reconcile reports an entry owned elsewhere
