---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shedadapters/bouncercarryover.go:101
    summary: The anchor-relative path error and the unreadable-ledger error of buildCarryOver have no test, and the refusal-spec carry-over row omits the path cause
  - id: F2
    severity: NIT
    class: consistency
    location: internal/loomcli/resume.go:207
    summary: The running row of the resume Long table lost one space of padding, so its description column no longer aligns with the other rows
  - id: F3
    severity: NIT
    class: consistency
    location: internal/discussionparser/carryover.go:216
    summary: The fenced-block and H2 scan is copied into locateOpenRisks and h2Headings beside decisionsInsertOffset, three copies of one boundary rule in a package
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`, and `_lyx/loom/rework/` does not exist, so there is no live rework round).
Nine commits, one per card.
`go build ./...`, `go vet` with the `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration` over every touched package all pass.

### [LOW] Untested carry-over error paths, and a refusal row that omits one of them

**Location:** `internal/shedadapters/bouncercarryover.go:101` (`anchorRelativePath`), `internal/shedadapters/bouncercarryover.go:39` (`buildCarryOver`), `contracts/specs/refusal-spec.md` row `carry-over entry refused`.
**Issue:** Card 2 makes "a path that does not lie under the anchor" an error, and `buildCarryOver` returns the ledger read and parse errors.
No test reaches either: `TestBouncer_ConvergedSettle_CarryOverFailureBlocksTheSettle` and `TestBouncer_Circling_AcceptCallsCarryOverBeforeSettling` fail through the seam's own error only, and `grep` finds no test naming `anchorRelativePath`, `buildCarryOver` or "does not lie under".
`PATTERN-refusal-way-forward` asks that a test reach each refusal.
The row's trigger lists the unreadable ledger but not the path outside the anchor, so a reader of the spec cannot tell that cause exists.
**Fix:** Add one row to the existing table in `bouncer_commit_test.go` whose `AnchorPath` is not an ancestor of `RunDir`, asserting the error names the path and the way forward and that Approve and Commit do not run.
Add the path-outside-anchor cause to the refusal row's trigger.

### [NIT] resume Long table misalignment

**Location:** `internal/loomcli/resume.go:207`.
**Issue:** The `running` row now has ten spaces before its description where every other row of the table has the description at column 20 (the previous version had eleven), a side effect of the table rewrite for card 9.
**Fix:** Restore the eleventh space.

### [NIT] Fence and H2 scan written three times

**Location:** `internal/discussionparser/carryover.go:216` (`locateOpenRisks`), `internal/discussionparser/carryover.go:339` (`h2Headings`), `internal/discussionparser/append.go:97` (`decisionsInsertOffset`).
**Issue:** Card 1 says the marker scan uses `decisionsInsertOffset`'s fence rule.
Each of the three functions repeats the same `` ``` ``/`~~~` toggle and `## ` test, so a change to what counts as a fence must land in all three or the splice self-check and the section locator disagree about where `## Open risks` ends.
**Fix:** Extract one helper that yields each line with its in-fence flag and use it from all three.

## Focus verdicts

Each focus question was settled against the code:

- Cards 1 to 3 hold. `discussionparser` alone renders, sanitizes and splices the entry (`renderCarryOver`, `sanitizeCarryOverField`, `checkCarryOverSplice`), the Bouncer passes `discussionparser.CarryOver` data only, and the seam runs before Approve on the CONVERGED branch of `settle` and before `settleCirclingAccept` on a pending accept.
  The skip seam never calls it.
  `carry_over` equals `segment` on all three shipped Bouncer rows, pinned by `TestBouncerRowsCarryOverTheirOwnSegment`.
- Cards 2 and 9 hold.
  The Bouncer's Stuck-path refusals (circling decision unreadable, accept already settled, escalation record unreadable or unwritable), the carry-over refusal and `circlingWayMalformed` keep `lyx loom start`.
  Only `escalationReason`, the circling verbs' output and wording, `circlingWayDecided`, the escalation stencil, the batten hints and the docs move to `lyx loom resume`.
  A failed CONVERGED settle is cleared and re-seeded by `lyx loom start` on re-entry, which the card states, so the refusal text "to re-run the settle" follows the plan.
- Card 5 holds.
  Round 1 renders byte-for-byte the previous rule.
  From round 2 on, `CONTINUE` needs a BLOCKING finding or a gating finding mapped to a key in the earlier-open list, and a parse-error line in that list sends the judge back to round 1's rule.
  `circlingEvidence` and the recurring-key list are untouched.
- Cards 6 to 8 hold.
  The six segment keys and `fix_start` are in `Config`, validated in `LoadConfig`, defaulted by `template.yaml`, resolved in `ResolveReview`, wired by row name through `Env.RowReviewModels` and `Env.FixStart`, and reach `RunOpts` unchanged through `BurlerProducer`.
  Under `after-review`, `decideRound` is shared with `join`, and a fixer that never started is a nil `Handle`.
- Per-card mechanical checks: no card carries a `Rename` or `Delete` group (the plan's cards list only `Create`, `Edit` and `Uses`), so no type-specific rename script or `assert-no-callers` check applies, and every named `Create` and `Edit` target exists in the diff.
- Test economy: the new tests each pin a distinct behavior, and the added rows extend existing tables as the cards direct.
  F1 is the one uncovered behavior.
