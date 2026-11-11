```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour, the pinning test**

All rows below are in `stepRunSettings`, `internal/boardcli/verbs_test.go`.

1. **A beam's orch outside its beam, parent is the prime, gets a refusal naming `lym creel post ly:orch --expect-reply`.**
   Pinned by the `refuses …` rows with `wayForward: toPrimeOrch`: an entry outside the beam, a new entry under no project, a member moved out of the beam, an entry moved into the beam, a non-string project link, and the beam itself.
   Each row also asserts the strand name appears in the message.
   If `refuseBeamOrchOutsideBeam` returned the old text, every one of these rows would fail.

2. **A nested beam's orch outside its beam is told to post to its parent beam's orch, `ly:bm:orch`.**
   Pinned by the new row "a nested beam's orch outside its beam" (`ly:inner:orch`, expects `lym creel post ly:bm:orch --expect-reply`).
   The added `inner` entry, a beam project under `bm`, is what makes `ParentBeamOf` return `bm`.
   Hardcoding the prime target would fail this row.

3. **The orch of another beam, whose slug has no board entry (`ly:other:orch`), gets the prime orch as target.**
   Pinned by the "the orch of another beam" row with `toPrimeOrch`.
   `ParentBeamOf` returns an empty string for an unknown slug, and the row confirms the refusal still comes out as the way-forward error.

4. **A role other than orch (`ly:bm:loom`) keeps the old refusal text, "the orch of the entry's beam".**
   Pinned by the "another role in the beam" row with `wayForward: "the orch of the entry's beam"`.

5. **The refusal still names the session.**
   Pinned by the `strings.Contains(message, tt.strand)` check on every row.

6. **Success paths are unchanged: the beam's orch writing inside its beam, and `ly:bm:orch-2`.**
   Pinned by the existing `runJSON(t, 0, …)` calls before the table.
   The diff touches none of this.

7. **`ParentBeamOf` returning an error is propagated from `refuseBeamOrchOutsideBeam`.**
   No row pins it. The branch is a pass-through of a board-load failure, and the neighbouring `BeamUnder` error branch has no row either.
   I raise no finding: it is an internal propagation with nothing to defend.

8. **The orch-role prose, the `doc.go` lines and the doc comments.**
   Static prose, exempt.

**Findings:** none.
- Test economy: the new row and the `wayForward` column extend the existing table-driven step. No new test function duplicates coverage, and the diff adds no assertion on static prose.
- I saw no redundancy-report entry that applies, and the fork's context lists none for these rows.

**Tool calls:** 0. The inherited context already held the diff and the test table.
