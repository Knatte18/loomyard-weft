---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/loomshed/prrework.go:1
    summary: New and changed Go comments across the diff are hard-wrapped at a fixed column mid-sentence, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/loomshed/prrework.go:33
    summary: PendingRejection's doc comment cites battenshed.ChildApproval, which card 15 renamed to ChildDecision
  - id: F3
    severity: LOW
    location: internal/landingshed/prgate.go:57
    summary: PR-Gate's three Done exits skip cancelErr, although card 7 says every exit consults it and the diff's own new Publish Done exits do
  - id: F4
    severity: LOW
    location: contracts/stencils/loom/loom-template-rework.md:55
    summary: The rework stencil lets a finding be covered by a coverage-file note instead of a card, contradicting card 8 and letting a round append zero cards that Go then rejects
  - id: F5
    severity: NIT
    location: internal/loomshed/prrework.go:158
    summary: The finish doc comment reads "finish is steps 6" and omits the absent-record path that also calls it
  - id: F6
    severity: NIT
    location: internal/battencli/wire.go:456
    summary: ReadDecision's comment claims approval-first matches the gate's precedence, but the gate prefers the matching record, so a stale approval beside a matching rejection resolves differently
  - id: F7
    severity: NIT
    location: docs/overview.md:415
    summary: The batten paragraph still says the operator runs only lyx loom approve before batten resumes the child on either decision
---

Review range: `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` = `main` read from `_lyx/shed/publish-review-fixback/status.json` (merge base `2cb4ac68`).
Substrate: `go build ./...`, `go vet ./...` and `go test ./...` pass.
`go test -tags integration` over fabricengine, landingshed, battencli, loomcli, loomshed and websterengine passes, and so does `go vet -tags smoke ./...`.

Per-card mechanical check: the plan's only `Delete` group is card 12's `internal/landingshed/publish_awaiting_test.go`.
assert-no-callers over the helpers that file alone defined (`publishOutcomeFor`), and over the symbols the card removed (`awaitingOrCancelled`, `checkApproval`, `newApprovalFixture`), finds zero remaining references.
Card 16 is `Prosa` and has no mechanical check.
No card carries a `Rename` group.

### [MEDIUM] Fixed-column-wrapped comments in new and changed Go code

**Location:** file comments and doc comments across the diff, including
`internal/loomshed/prrework.go` (file comment, `appendOnlyViolation`, `roundFor`),
`internal/landingshed/prgate.go` (file comment),
`internal/landingshed/rejection.go` (file comment, `WriteRejection`),
`internal/landingshed/approval.go` (changed file-comment sentence, `writeRecordAtomic`),
`internal/landingshed/deps.go` (the rewritten `ApprovalPath`/`RejectionPath` comment),
`internal/landingshed/doc.go` (the rewritten Publish sentences),
`internal/landingshed/publish.go` (file comment, the open-PR case comment, `refreshPullRequest`),
`internal/fabricengine/committedfile.go`,
`internal/loomcli/reject.go` and `approve.go` (file comments),
`internal/loomcli/wiring.go` (the `Rework` comment),
`internal/loomengine/rework.go` (`composeReworkPrompt`, the friction comment),
`internal/planparser/appendonly.go`,
`internal/shedrecipe/entries_prrework.go`, `entries_simple.go` (`prGateEntry`), `recipe.go` (`ReworkSpec`, `Rework`),
`internal/websterengine/fingerprint.go` (`RebaselinePlanFingerprint`),
`internal/loomrecipe/gatequiescence_test.go`, and the new test files' header and test doc comments.
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires semantic line breaks in every new or changed Go comment: one sentence per line, a further break only at an independent-clause boundary, no column limit.
These comments break at roughly column 100 mid-phrase (for example `does\n// everything Go owns`, `the round\n// record committed`, `holds it\n// awaiting a decision`), and several put two sentences on one line (`deps.go`, `doc.go`).
**Fix:** Rewrite each listed comment to one sentence per line, breaking inside a sentence only at an independent-clause boundary, and leave untouched legacy lines as they are.

### [LOW] Stale `battenshed.ChildApproval` reference

**Location:** `internal/loomshed/prrework.go:33`
**Issue:** `PendingRejection`'s doc comment says the caller fills it "the way battenshed.ChildApproval is filled".
Card 15 renamed that type to `ChildDecision` in the same diff, so the comment names a type that no longer exists.
A sweep of the tree for `ChildApproval`, `InnerRunDeps.ReadApproval`, `approvalActedFile` and `-approval-acted` finds no other stale reference.
**Fix:** Name `battenshed.ChildDecision`.

### [LOW] PR-Gate's Done exits skip cancelErr

**Location:** `internal/landingshed/prgate.go:57`, `:81`, `:105`
**Issue:** Card 7 says every exit of `PRGate.Call` consults `cancelErr` first.
The Awaiting and Stuck exits do, through `awaiting`/`stuck`, but the not-required, merged and matching-approval exits return `Done` directly.
A context cancelled after the GitHub query returned is then reported as Done.
The verdict is still truthful, since the decision was fully computed, which is why this is LOW.
It does differ from the card's wording and from the diff's own new Publish Done exits (after create, and in `refreshPullRequest`), which do consult `cancelErr`.
**Fix:** Route the three Done exits through a `done(ctx)` helper that consults `cancelErr` first, matching `awaiting` and `stuck`.

### [LOW] Rework stencil admits a finding with no card

**Location:** `contracts/stencils/loom/loom-template-rework.md:55`
**Issue:** Card 8 says every finding is covered by at least one new card.
The stencil adds "A finding that needs no code change still needs a card or an explicit entry in the coverage file stating why".
That lets the agent skip a card.
When every finding is judged to need no change, the agent appends zero cards, and `CheckAppendOnly`'s "no card was appended" rule makes the row Stuck.
The prompt therefore invites an outcome Go rejects.
**Fix:** Drop the escape clause, so that every finding is covered by at least one new card, as card 8 states.

### [NIT] `finish` doc comment

**Location:** `internal/loomshed/prrework.go:158`
**Issue:** "finish is steps 6" is ungrammatical.
The comment also describes only step 6, although the absent-record crash path calls `finish(false)` too.
**Fix:** Say it runs step 6, and that the crash path uses it without clearing.

### [NIT] ReadDecision's precedence claim

**Location:** `internal/battencli/wire.go:456`
**Issue:** The comment says the approval winning "match[es] the gate's own precedence".
The gate checks a *matching* approval first, and then a *matching* rejection, so a stale approval beside a matching rejection sends the gate to rework while `ReadDecision` reports the approval.
Both records can coexist only when `approve` and `reject` race, since each removes the other's record first.
The approval-first order is what card 15 specifies, so the code stands; only the rationale overstates.
**Fix:** Reword the comment: both records coexist only when the two verbs race, and the approval wins then, as it does in the gate when both match.

### [NIT] Batten paragraph names only `approve`

**Location:** `docs/overview.md:415`
**Issue:** "the operator runs `lyx loom approve` in the task worktree, batten resumes the child itself once per operator decision, approval or rejection alike" still names only `approve` as the operator's action.
**Fix:** Name `lyx loom approve` or `lyx loom reject`.

## Focus directives settled without a finding

- Rejection budget arithmetic: shedengine compares `episodeStuckCount(st.History, gate)` (the history before the current Stuck is appended) against `effectiveMaxBounces` = 5, so gate Stucks 1–5 route to PR-Rework and the 6th blocks.
  `reject` reads `Bounces` over the same pre-Stuck history and refuses at count ≥ 5, so every rejection `reject` accepts routes to PR-Rework, and no round the budget allows is refused.
  The gate never returns Done between rounds, so the episode does not reset, which makes the header's "five rejection rounds per task" correct.
- PR-Rework's absent-record branch: `approve` refuses at PR-Rework, and `reject` at PR-Rework writes a record, so only the post-`ClearRejection` crash or a hand-deleted record reaches the branch.
  The hand-deleted case routes to Webster and back to the gate, which awaits.
  `roundFor` and `committedRoundHeads` agree: an uncommitted leftover round for another head is skipped by the committed lookup and only raises the next round number.
- Anchor-relative joins: `LoomReworkDirRel` uses `filepath.Join`, `prrework.go` appends with `path.Join`, and `ScopedPathspec`'s `filepath.Join` plus `filepath.ToSlash` normalises the mixed separators to the slash form `FileAtRevision` needs on Windows; POSIX is slash-only throughout.
  A card absent at HEAD wraps `fs.ErrNotExist`, and `parseCardFile` maps it to "card file not found".
- Publish split: the surviving "Step 3a" is the former "Step 3b" (the status commit), renumbered after the approval step went, so the numbering is coherent.
  The title/body comparison uses the same summary fields the create call sends, and the edit failure takes the same transient-or-Stuck split.
- Files outside any card's target list: `gatequiescence_test.go` must admit `PRRework` as a gated writer row, or the new recipe row fails that test.
  `shedrecipe/registry_test.go` must list the two new registry keys, or `TestRegistry_ShipsExpectedEntries` fails.
  Both edits are correct and needed.
