---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/bouncer/bouncer-template-escalation.md:31
    summary: The shared escalation brief now carries darn wording in every loom run, so loom's brief changed although its decision record always exists
  - id: F2
    severity: LOW
    class: scope
    location: internal/loomcli/darnwiring.go:121
    summary: The warn-and-continue branch of renameRetiredDarnRows, taken when a live driver holds the run lock, has no test
  - id: F3
    severity: NIT
    class: consistency
    location: internal/loomcli/darnwiring.go:201
    summary: wireDarn discards the isBouncer result of BouncerRunSubdir, so a missing run_subdir would silently point ReviewDir at the reviews root
---

### [LOW] Shared escalation brief changed for loom runs

**Location:** `contracts/stencils/bouncer/bouncer-template-escalation.md:31`, `contracts/stencils/claims_test.go:345`
**Issue:** The brief is one stencil for every review segment.
The diff rewrites step 1 to "When the decision record exists, record each design call…" and adds "A darn run has no decision record…".
A loom run always has a record, so its parent fork now reads a condition that is never false and a sentence about another recipe.
The focus directive asked whether the darn fix leaves loom's brief unaltered; it does not.
The darn behaviour itself is correct: the new test shows the brief renders and no record is written.
**Fix:** Keep the wording, or make the darn branch a marker filled only by the darn wiring (empty in loom) so loom's rendered brief is unchanged.
If the shared wording stays, accept this as a deliberate trade-off and say so in the summary, which currently states only the darn side.

### [LOW] Busy-lock path of `renameRetiredDarnRows` is uncovered

**Location:** `internal/loomcli/darnwiring.go:121`
**Issue:** The summary claims a live driver holding the run lock is left alone with a warning.
`TestRenameProducers/a held run lock refuses busy and leaves the file` covers `RenameProducers` returning `ErrShedBusy`.
The `errors.Is(err, shedengine.ErrShedBusy)` branch in loomcli, which turns that refusal into a warning and lets the verb continue, is exercised by no test.
`TestArmAt_RenamesRetiredDarnRows` has only the unlocked rows.
This branch is the one that decides whether `status` and `pause` still work against a run whose old-binary driver is alive, and a regression that propagated the error would refuse them.
**Fix:** Add a row to `TestArmAt_RenamesRetiredDarnRows` that holds the run lock, then asserts `armAt` returns no error and the status file still names the old row.

### [NIT] Ignored `isBouncer` result in `wireDarn`

**Location:** `internal/loomcli/darnwiring.go:201`
**Issue:** `reviewSubdir, _, err := loomrecipe.BouncerRunSubdir(...)` ignores the second result.
For the embedded recipe the row is always a Bouncer with a `run_subdir`, so this cannot occur today; if the recipe lost the key, `ReviewDir` would become the reviews root and the Describe stencil would read the wrong directory without an error.
**Fix:** Return an error when the second result is false, as the circling verb does for a non-Bouncer row.

## Checked and found sound

- `RenameProducers` rewrites `current_producer`, `pause_before`, `pause_after` and every history producer, reads without the run lock, takes no lock and writes nothing when no retired row is named, and refuses with `ErrShedBusy` under a held lock; each is pinned in `TestRenameProducers`.
- `armAt` applies it for darn runs before `loadRouting`, so the lightweight verbs see the new name; a loom run's file is left alone (`TestArmAt_RenamesRetiredDarnRows`).
- No `Darn` row reference remains in code, recipe, stencils, docs, refusal spec or help; the only remaining uses are the deliberate retired-name map and its tests.
  The interrupt policy moved to `NameDarnWrite`.
- Recipe edges: `Darn-Bouncer` `on_done` goes to `Describe`, then `Publish`; `review_max_bounces: 0` routes `Darn-Write` to `Publish`; `PR-Gate` `on_stuck` is `Darn-Write`.
- `wireDarn` replaces `DescribeSpec` with `DarnDescribeSpec`.
  The stencil's `round-<N>-circling-decision.md` and `decision: accept` match `shedadapters`' file name and frontmatter, and the review directory matches where `RecordCirclingDecision` writes.
  The shared composer keeps `DescribeSpec`'s output and error prefixes.
- `NewDarn` stamps `verify_attempts` on both gates (`darnrecipe_test.go`); the recipe's `attempts: 3` values are documented placeholders.
  The `review_max_bounces` default of 1 is consistent in `template.yaml`, `darnengine` docs, `docs/overview.md`, recipe comments and the darn test env; loom's own default of 3 is unrelated.
- `TestPublish_OpenPullRequest` exists in `internal/landingshed/publish_test.go` and pins the refresh of an open pull request's title and body.
  Publish is shared by loom and darn, so the board entry's request to check loom's rejection path is satisfied by that test and by the summary-spec correction.
- `TestEscalation_DarnPerchBudgetSpentEscalatesThenAcceptReachesDescribe` is new coverage for the spent budget and duplicates no existing test.
  The test budget file was raised by one for each of `landingshed`, `loomcli`, `loomrecipe` and `shedengine`, the packages that gained a test.
