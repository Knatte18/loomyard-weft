---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/03-fabric-pair-anchored-push.md
    summary: Card 3 changes the PushAnchored signature but its doc.go edit names only the anchored entry-point paragraph, leaving the two-argument PushAnchored(l, opts) mention in the list of l-in entry points stale
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/03-fabric-pair-anchored-push.md
    summary: Acceptance criterion that batten's status seam still pushes only the records side has no assertion, because the existing PushAnchored success test never leaves an unpushed code-side commit
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/02-gitrepo-has-unpulled.md
    summary: Card 2 Uses lists Repo.Fetch although its Intent has HasUnpulled read the remote-tracking ref as last fetched and never call Fetch
---

## Focus directive outcomes

Both directive entries were reached; neither is a defect.

- **Pinned run-bound list.**
  Card 2 is the only card that adds a `Repo` method calling `r.runChecked` (`HasUnpulled`), and it edits `cmd/lyx/gitrepoboundary_test.go` in the same card, adding the method to `gitrepoPinnedRunBoundMethods`, so `TestGitrepoBoundary_PinnedRunCallSites` stays green at card 2's commit.
  Card 3's retry helper reuses `Repo.Fetch`, which is already on the list, and adds no `Repo` method; cards 1, 4 and 5 touch no `internal/gitrepo` file.
  `HasUnpulled` introduces no new `gitexec` call expression, since it goes through `runChecked`, so the guard's second assertion (exactly two `gitexec.Run`/`RunGit` expressions) is untouched.
  `go test ./...` is therefore green at each card's commit as far as this guard goes.
- **`PushAnchored` reached directly.**
  The absorbing lock with a bounded wait and the `ErrPushLockBusy` expiry are reached for `PushAnchored` by the table-driven lock test, which runs over both entries.
  The fetch-checked retry is reached by the scenario test's retry step, which runs `PushAnchored` against a records remote that declines once.
  A rejection that stays is reached by the existing `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejectedUnwrapped`, kept as the bare-sentinel pin.
  Each is a row or step in an existing test, which `PATTERN-test-economy` counts as coverage.
  Note on that last test: a diverged remote still ends in the bare sentinel whether or not the retry ran, so the test pins the outcome and not "no second push", and the outcome is the only observable the decision record asks for.

### [LOW] Stale PushAnchored signature in fabricengine/doc.go

**Location:** `_lyx/plan/03-fabric-pair-anchored-push.md`, Intent paragraph on `doc.go`; `internal/fabricengine/doc.go` where it lists "`PushAnchored(l, opts)`, `MergeStateActive(l)`, …".
**Issue:** Card 3 names `doc.go`'s "anchored entry-point paragraph" and the "human-decidable condition rather than retrying" sentence, but `doc.go` also lists `PushAnchored(l, opts)` among the vocabulary-neutral entry points.
After the card the signature has a third parameter, and the docs-in-the-same-commit rule means a stale signature is left behind unless the card says to touch that line.
**Fix:** Name that list entry in the card's `doc.go` instruction, so it reads `PushAnchored(l, opts, lockWait)` and gains `PushPairAnchored(l, opts, lockWait)`.

### [LOW] Batten's records-only push has no test

**Location:** `_lyx/plan/03-fabric-pair-anchored-push.md`, Tests section.
**Issue:** The decision record's acceptance criteria say "Batten's status seam still pushes only the records side", and card 3 keeps batten on `PushAnchored`.
The only test that could catch a `PushAnchored` that also pushed the code side is `TestPushAnchored_PushesAndRecordsBranchPush`, which commits on the weft alone, so a code-side push would go unseen.
`battencli`'s own tests use a stub `Push` closure.
**Fix:** Extend that existing test, as an assertion and not a new test: also make a raw commit in the pair's warp worktree, and assert after `PushAnchored` that the warp bare remote still sits where it did.

### [NIT] Card 2 lists a Use that HasUnpulled does not call

**Location:** `_lyx/plan/02-gitrepo-has-unpulled.md`, Uses.
**Issue:** `internal/gitrepo#Repo.Fetch` is listed, but the Intent has `HasUnpulled` run `rev-list --count HEAD..@{u}` against the ref as last fetched; the fetch happens in card 3's retry helper, and card 2's test fetches through a fixture.
**Fix:** Drop `Repo.Fetch` from card 2's Uses; keep `Repo.HasUnpushed` as the mirror it sits beside.
