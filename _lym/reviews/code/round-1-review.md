---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_stalemate_test.go:170
    summary: No test pins that a round with neither a target id nor a post-fix id counts as changed, the guard that alone keeps a missing overlay target from reading as a stalemate.
    fix: "In the stalemate table of internal/shedadapters/bouncer_stalemate_test.go, add a row beside RoundWithNoPostFixIdCountsAsChanged named RoundWithNoTargetIdCountsAsChanged, with stalemateRounds 2, earlier []stalemateRound{{target: \"\", post: \"\"}}, lastTarget \"T0\", tree \"T0\" and want shedengine.Stuck. In internal/shedadapters/bouncer_verify_test.go, add the matching case for fixLeftTargetUnchanged: a round-2 snapshot record with TargetID and PostFixTargetID both empty must not read as a fix that left the target unchanged, next to the existing tree-1/tree-1 case at line 173."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/roundsnapshot_test.go:51
    summary: No row pins that a round whose overlay target matches no file still writes its snapshot record, with an empty target_id.
    fix: "Add a row to the table in internal/shedadapters/roundsnapshot_test.go whose overlay TargetSnapshot paths name a file that does not exist under the root. Assert that the round-1 record is written with FixScope overlay, the configured TargetPaths and TargetID equal to \"\"."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedadapters/targetsnapshot.go:32
    summary: The doc comment on TargetSnapshot.Paths still calls the paths worktree-relative, though the review snapshot now roots them at the anchor path.
    fix: "In internal/shedadapters/targetsnapshot.go, change the Paths field comment to read \"Paths are the target paths relative to WorktreeRoot, files or directories.\""
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:33
    summary: The blast radius says bouncer, but the new snapshot root also changes the review row's recorded round-start target id and its round changes file, which decide incremental review rounds.
    fix: "In _lym/landing/summary.md, replace the blast radius word bouncer with one word that covers both the review row and the Bouncer, such as review."
---

### [MEDIUM] The guard that stops an empty target from causing a stalemate has no test

**Location:** `internal/shedadapters/bouncer_stalemate_test.go:170`

**Issue:** With this diff, an overlay target that matches no file makes `ID` and `Take` fail.
The round's record then holds no `TargetID` (`burlerreview.go:230-232`) and no `PostFixTargetID` (`bouncerstalemate.go:33-37`).
The change description and commit f635cccab both claim that a missing target can never produce a stalemate.
That claim, which is the board entry's second expectation, depends on the `record.TargetID == ""` guard in `roundUnchanged` (`bouncerstalemate.go:55`) and in `fixLeftTargetUnchanged` (`bouncerverify.go:185`).
Delete either guard and two empty ids compare equal, so the round reads as unchanged, yet no test fails.
`RoundWithNoPostFixIdCountsAsChanged` uses target `T0`, so it pins only the inequality check.

### [LOW] A round over a missing target writing an empty target_id has no test

**Location:** `internal/shedadapters/roundsnapshot_test.go:51`

**Issue:** When `Take` fails, `recordRoundStart` logs a warning and writes the round's record with an empty `TargetID` (`burlerreview.go:229-232`).
Until this diff that path needed an unreadable file. Now any overlay target that matches no file reaches it.
The table at `roundsnapshot_test.go:49-51` covers an accepted id, a kept first id and a nil snapshot, but not a `Take` that fails.

### [LOW] The doc comment on TargetSnapshot.Paths no longer describes what the field holds

**Location:** `internal/shedadapters/targetsnapshot.go:32`

**Issue:** `Paths` is documented as "the worktree-relative target paths".
After the diff, `reviewTargetSnapshot` (`entries_burler.go:59`) sets `WorktreeRoot` to `Env.AnchorPath`, so on a hub anchored to a subpath these paths are relative to the anchor.
The `WorktreeRoot` field's own comment ("the absolute root the overlay target paths are relative to") is still accurate, and the summary deliberately keeps the field's name.
The `Paths` line is the one left wrong.

### [LOW] The blast radius leaves out the review row

**Location:** `_lym/landing/summary.md:33`

**Issue:** `reviewTargetSnapshot` builds the snapshot for both the review row (`entries_burler.go:182`) and the Bouncer (`entries_bouncer.go:312`).
The new root therefore also changes:
- the round-start `target_id` that `recordRoundStart` records;
- the changes file that `recordRoundChanges` writes (`roundcontext.go:283-318`).

That changes file decides whether a review round is incremental, and that in turn decides whether its new findings are held to BLOCKING.
The blast radius `bouncer` covers only half of what the change can break.
The `two-way` door holds. The change alters no record format, deletes no data and pushes nothing outward. An in-flight round recorded under the old root reads as changed, which is the safe direction.

## Rejected

- code-scope, LOW scope ("fail loudly" only partly built, since callers log at Warn and go on): rejected.
  The error now comes from the point where the empty hash used to be formed.
  The callers' Warn-and-continue is how they already handle every failed id read, and it satisfies the entry's stated contrast, "not count as an unchanged target".
  The summary records this as a deliberate design decision.
  Failing the round instead would stop a run because of the stalemate check, and the entry does not ask for that.
