---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/01-weft-lock-dir.md:40
    summary: Card 1 plans a hubgeom parent_test.go case and geometry step (b) that both assert the ResolveParent lock-directory error on the same behavior
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-add-adopts-live-pair.md:72
    summary: Card 4's fabriccli row for an origin holding only the weft duplicates the fabricengine origin-weft-only adopt row, and the both-tips CLI row already pins the add wiring
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/06-finalize-config-notice.md:50
    summary: Card 6's fake-seam row "an already-up-to-date merge queues the notice again" takes the same pushParent Done path as the non-empty row and repeats the integration step's second Finalize
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/03-reconcile-adopts-origin-weft.md:21
    summary: Card 3 states the bound of the archive-guard bypass but not what stays behind when adoptWeftWorktree fails after the resolver created the local branch from origin
---

## Review

Target: `_lyx/plan/` cards 1 to 6 and the overview, judged against `_lyx/discussion/decision-record.md`.
I read every card, the overview and the decision record, then checked each named test file and named test against the tree.

Settled from the focus directive:

- **D1 weft-creating helpers.** Card 1 carries `createWeftWorktree`, `createWeftWorktreeDroppingRuns`, `adoptWeftWorktree` and Add's own adopt branch.
  These are the four `git worktree add` sites for a pair's weft in `internal/fabricengine` (`weftwiring.go`, `forkrecords.go`, `reconcile.go`, `add.go`); `boardweft.go` mints the board worktree, which D1 does not list.
  The D7 origin-adopt path needs no further card-1 edit: reconcile reaches it through `adoptWeftWorktree`, Add through the same adopt branch card 1 edits, and `checkout` creates no worktree.
  Card 4 says explicitly that card 1's adopt-path edit creates `.weft/` there.
- **D3 premise.** `ReadOriginFor` today stats the record and the lock directory in one loop and answers "no record" for either; card 1's change matches.
- **D6 and the acceptance criteria.** Every `add` bullet and criterion traces to a card-4 row, including the skip-option row (`TestAdd_SkipPushSkipsLeftoverProbes`), the archive-tag-plus-local-weft row, the fast-forward and true-divergence rows, and the failed-Add-keeps-fast-forward row.
  `TestAdd_WarpLeftoverRefusedAtPreflight` uses a plain `Remove`, which leaves an archive-covered weft, so the refusal card 4 says is unchanged stays unchanged.
- **D4.** Cards 5 and 6 carry the HubWide filter (`configreg.Modules()` with `HubWide` false), the diff read before the catch-up merge-in, the failed-read notice, `QueueNotice` errors and `queued=false` leaving the verdict, the resume re-queue and the empty-diff silence.
- **Granularity, card 4.** `weftLeftover`, `leftoverKind` and the probes are private supporting parts of Add's pre-flight, and `ErrRemoteLeftover` is that pre-flight's one error; bundling them is right.
- **Attack surface.** Cards 2, 3 and 4 each state what the change lets through and the origin-untouched bound; card 4 and card 2 state rollback of a created branch.
  Card 3 is the one gap, F4.
- **Verify coverage.** Every package a card edits or creates a test in (`fabricengine`, `fabriccli`, `hubgeom`, `landingshed`, `loomcli`) is in the `go vet -tags llm`, `-tags integration` and `-tags tmux` runs, and `go test ./...` covers the untagged tier.
- **Test paths after the prunes.** Every existing file a card edits exists, and the named tests and steps are there (`TestGeometryOverRealHub`, `TestFinalize_OverRealHub` with `newFinalizeAt`, the `AddLeftoverWeftIsBarePreflightError` step, `landingdeps_markdone_test.go`, `landingdeps_test.go`).
  The two files absent today, `internal/fabriccli/adoptremote_integration_test.go` and `internal/loomcli/landingdeps_integration_test.go`, are created by cards 2 and 6.

### [MEDIUM] Card 1 asserts the lock-directory error twice

**Location:** `_lyx/plan/01-weft-lock-dir.md`, Tests, first bullet and step (b).
**Issue:** The `parent_test.go` case removes `.weft` under `hubWithOrigin` and asserts `ResolveParent` returns the lock-directory error naming `lyx fabric reconcile`.
Step (b) in `geometry_integration_test.go` deletes `.weft` on a real pair and asserts the same `ResolveParent` error, then runs `reconcile` and asserts the heal.
Both reach `ReadOriginFor`'s new error through `ResolveParent`.
D8 requires the real-wiring test, which step (b) is, so the unit case adds no coverage.
**Fix:** Drop the `parent_test.go` case and remove `internal/hubgeom/parent_test.go` from the card's `Edit` list.

### [MEDIUM] Card 4 pins the origin-weft-only adopt twice

**Location:** `_lyx/plan/04-add-adopts-live-pair.md`, Tests, the `add_remoteleftover_integration_test.go` and `adoptremote_integration_test.go` bullets.
**Issue:** The converted `TestAdd_UnarchivedWeftLeftoverRefusedAtPreflight` row already asserts the weft adopted tracking origin, carrying the pushed commit, with the warp forked from HEAD.
The `adoptremote_integration_test.go` bullet adds "origin holding only `<slug>-weft`: weft adopted tracking origin, warp forked from HEAD", the same behavior through the CLI.
The both-tips row in that bullet already drives `add`'s adopt path through `lyx fabric add`, which is what D8 asks.
**Fix:** Remove the origin-holding-only-`<slug>-weft` clause from the `adoptremote_integration_test.go` bullet and leave the both-tips row.

### [LOW] Card 6's already-up-to-date fake row repeats another test

**Location:** `_lyx/plan/06-finalize-config-notice.md`, Tests, `finalize_test.go` bullet.
**Issue:** An already-up-to-date parent merge returns no error, so it takes the same `pushParent` Done return as the non-empty-diff row, and the fake row asserts nothing that row does not.
The resume behavior itself is pinned over a real landing by the `an already landed parent is idempotent` step's second Finalize.
**Fix:** Remove the "an already-up-to-date merge queues the notice again" clause from the `finalize_test.go` bullet.

### [LOW] Card 3 is silent on a fetch-created branch left behind by a failed adopt

**Location:** `_lyx/plan/03-reconcile-adopts-origin-weft.md`, Intent, the resolver-error bullet and the Bound paragraph.
**Issue:** The resolver can create the local weft branch from origin and `adoptWeftWorktree` can then fail; Reconcile has no rollback, and the card says only that a resolver error blocks wiring.
The bound covers what the bypass pushes and replaces, not what remains locally.
**Fix:** State that a branch the resolver created stays in place when the adopt fails, is recorded in the verb's `*Mutations` record, and is adopted as a local branch by the next reconcile, with origin untouched.
