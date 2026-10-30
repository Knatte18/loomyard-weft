---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/14-pr-review-session.md:66
    summary: "Card 14 lists only parentdirective and editdirective as the new landingshed imports, but ensureSession also needs internal/stencil to fill the stencil, which the allowlist lacks, so TestToldGeometryInvariant_AllowlistOnly fails at card 14"
    fix: "In card 14's Intent, rewrite the closing allowlist paragraph so that `landingshedAllowedImports` gains three entries: `github.com/Knatte18/loomyard/internal/parentdirective`, `github.com/Knatte18/loomyard/internal/editdirective` and `github.com/Knatte18/loomyard/internal/stencil`, the last one for the `stencil.Fill` call that fills the `landing-template-pr-review` markers, as `internal/mergeresolve/spec.go` does. Also say that ensureSession builds the skills list with a plain append of `Deps.LanguageSkills` after `scribe:prose` and `scribe:conversation`, so it never imports `internal/languageconfig`. No target changes: `landingshedAllowedImports` and `TestToldGeometryInvariant_AllowlistOnly` are already under card 14's Edit."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/plan/13-pr-review-row-waits.md:61
    summary: "On a re-entry with a record, card 11 step 6 reads Status and the wait's first wake polls Status again, two reads in the same second, which breaks the once-a-minute bound and the card's own at-most-once-a-minute test row"
    fix: "In card 13, change `wait`'s inputs so that `Call` hands it the `pullrequest.Status` it read in card 11's step 6 and the injected-clock time of that read, and so that the wait's poll clock starts at that time. The first wake then uses the handed status as its poll result and polls again only a minute later, or at once on the continue-flag transition. When step 6 made no read (a first entry, or a kept ClosedForContinue record), the first wake polls. In `TestPRReview_Wait`, add a row: a re-entry with a recorded open pull request shows exactly one Status call in the first minute of the injected clock, and make the row seeded with `Continue` before `Call` count Status calls from `Call` onward, so step 6's read is the one poll of that minute. Name the rule in the overview's Shared Decisions bullet `One immediate poll on the continue flag`, or in a bullet beside it, so every poll beyond the decision record's bound is recorded in one place."
  - id: F3
    severity: LOW
    class: consistency
    location: _lym/plan/11-pr-review-row-opens.md:94
    summary: "Card 11 rewrites only header comments that call the row a stub, but loom-recipe.yaml and darn-recipe.yaml also carry comments saying PR-Review never returns Stuck, which the row's Halt falsifies"
    fix: "In card 11's Recipes paragraph, name two more rewrites: the header sentence in `contracts/recipes/loom-recipe.yaml` that reads `PR-Review and Friction-Reflect are outside the escalate-to-a-human set: each also carries no on_stuck, but each always returns Done and never Stuck`, and the header sentence in `contracts/recipes/darn-recipe.yaml` that reads `PR-Review, like Friction-Reflect, carries none either, because it never returns Stuck`. Each is rewritten so that Friction-Reflect keeps the never-Stuck claim and PR-Review is said to carry no on_stuck because its Stuck is always a Halt. No target changes: both files are already under card 11's Edit."
---

### [BLOCKING] Card 14's allowlist addition omits internal/stencil

**Location:** `_lym/plan/14-pr-review-session.md:66`

**Issue:** `landingshedAllowedImports` in `internal/landingshed/seam_enforcement_test.go` lists no `internal/stencil`, `internal/parentdirective` or `internal/editdirective`, and `scankit.AssertImportAllowlistNoStale` fails on an unlisted import.
Card 14 adds the last two and says `TestToldGeometryInvariant_AllowlistOnly` passes at that card.
Its `ensureSession` fills `landing-template-pr-review` with both directives and the path markers "as `internal/mergeresolve`'s spec does", and `internal/mergeresolve/spec.go` fills the template with `stencil.Fill` from `internal/stencil`.
No landingshed production file imports `internal/stencil` today: Describe and the fix review hand seat tables to `seatengine`, which fills inside its own package, and an unawaited session has no such route.
So card 14's gate fails on the unlisted import even though the card claims the opposite.
The other cards in the focus are sound on this point: card 8 lists `internal/githubclient`, card 11 lists `internal/pullrequest`, and card 13 adds no import beyond those and `internal/gateslot`, which is listed.

### [MEDIUM] Entry reads Status twice in the first second

**Location:** `_lym/plan/13-pr-review-row-waits.md:61`

**Issue:** Card 11's step 6 reads `Status` whenever a record exists, then card 13's `Call` hands the record to `wait`, whose first wake reads `Status` again.
Both reads are a GitHub poll in D9's sense, so a re-entry (a crash, a resume, a Finalize reroute, the end of a pause) polls twice within one second.
That is a second extra poll beyond the decision record's bound, and the overview records only the continue-flag poll as a departure.
It also contradicts the card's own test row, which asserts GitHub is polled at most once a minute of the injected clock across a record seeded with `Continue` before `Call`: that row starts from a record, so step 6 and wake 1 give two `Status` calls at the first instant.
Neither the card nor card 11 says where the wait's poll clock starts.

### [LOW] Recipe header comments that say PR-Review never returns Stuck

**Location:** `_lym/plan/11-pr-review-row-opens.md:94`

**Issue:** Card 11 rewrites "every header comment calling the row a stub that always returns Done".
`contracts/recipes/loom-recipe.yaml` has a header sentence saying PR-Review and Friction-Reflect "each always returns Done and never Stuck", and `contracts/recipes/darn-recipe.yaml` has one saying PR-Review "never returns Stuck".
Neither calls the row a stub, so the card's wording does not reach them, and both turn false once the row halts.
Both files are already under the card's Edit, so the fix is naming the sentences.

## Focus verdicts

- Allowlist: card 8, card 11 and card 14 each list and test the allowlist under Edit and state their added imports; only card 14's list is incomplete (F1).
- Import guard: every package the plan makes import `internal/pullrequest` is on card 16's allowed list: `internal/landingshed`, `internal/pairteardown`, `internal/loomcli` and `internal/testkit/envkit`. The existing `.Issues.Edit(` call in `internal/hubissues` and the read-only `.PullRequests.List(` calls are allowed as the card says. Not a defect.
- `ClosedForContinue`: card 13's closed-and-`ClosedForContinue` Done verdict is reached by a named row that starts from a kept record, and card 11 has the matching `Call` row. A re-entry after a crash between recording `ClosedForContinue` and returning Done keeps the operator's ending, because step 6 keeps the record while the head equals `GatedHead`. Not a defect.
- Mail to the agent: `verifyGate.check` returns a reason that names no tier and speaks of merging the parent branch and of resume, so card 13's `checkHead` is needed. Its wording names the failing tier and the log path as the decision record requires and suits an agent that keeps working in its session. Not a defect.
- Docs: `README.md`'s Landing bullet and `internal/loomrecipe/doc.go`'s sentence are both on card 11's target list with their rewrites stated. The recipe header sentences are F3.
- Poll cadence: the continue-flag poll is recorded in the overview and pinned by a named row, and the entry read before the first wake is neither recorded nor pinned (F2).

## Rejected

- The continue-flag poll's departure from D9's once-a-minute bound has no `Added after Discussion` entry: round 2 raised it and its fixer recorded the departure in the overview's Shared Decisions, which was the proposed fix, so it is settled and not raised again.
