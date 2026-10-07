---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-add-adopts-live-pair.md:72
    summary: the fabriccli add row asserts origin's tips are unchanged after the push, but Add commits and pushes an origin record onto an adopted weft that carries none, so the weft tip moves
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/03-reconcile-adopts-origin-weft.md:27
    summary: the reconcile bound leaves a resolver-created local weft branch behind on adopt failure without saying that add's live-pair rule then reads it as proof of a live pair, unlike checkout and add which roll the branch back
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/02-weft-branch-resolver-checkout.md:1
    summary: card 2 changes checkout to adopt an origin weft and to fail on an unreachable origin but edits no doc.go, while cards 3 and 4 document the same change for reconcile and add
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/04-add-adopts-live-pair.md:65
    summary: the lyx fabric add help text in internal/fabriccli/fabric.go says the new weft branch is forked from HEAD, which is false for a live pair, and no card edits it
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/06-finalize-config-notice.md:50
    summary: the nil seams change nothing row duplicates every existing Finalize test, which already runs with nil ConfigChanges and Notify
---

### [MEDIUM] The add CLI row asserts origin's tips unchanged, which contradicts the adopt rule

**Location:** `_lyx/plan/04-add-adopts-live-pair.md:72` (the `internal/fabriccli/adoptremote_integration_test.go` row), against decision record D6 and its acceptance criterion.
**Issue:** The row builds origin's weft from a scratch clone holding a run record, not an origin record, and asserts "origin's tips are unchanged after the add's push".
The same card's Intent says an adopted weft with no record gets one written and committed naming the acting worktree, and D6 says the weft is then pushed.
So the weft tip moves by one commit on that row.
The decision record's acceptance criterion says the add's push is a fast-forward, not a no-op.
Only a scratch weft that already carries an origin record leaves its tip unchanged, but that case is the `origin_integration_test.go` row, so making the CLI row carry one would duplicate it.
**Fix:** Change the row's last assertion to match the acceptance criterion.
The warp tip is unchanged, and origin's weft tip is a fast-forward descendant of the tip the scratch clone pushed, carrying the origin record commit.
State in the row that the scratch weft carries no origin record.

### [LOW] The reconcile bound omits what a leftover origin-created weft branch means to add

**Location:** `_lyx/plan/03-reconcile-adopts-origin-weft.md:27`
**Issue:** This answers the focus question of what remains locally when `adoptWeftWorktree` fails after the resolver created the branch.
The Intent states the residue: the branch stays, it is in the `*Mutations` record, the next reconcile adopts it, and origin is untouched.
It does not state that a surviving local `<slug>-weft` branch is exactly what card 4's live-pair rule reads as proof of a live pair, which skips the archive-tag guard, and which D6's bound rests on `Remove` always deleting.
For an archive-covered weft, a failed raw-worktree adopt, followed by the operator removing that worktree, leaves a local branch that makes a later `lyx fabric add <slug>` adopt origin's diverged warp.
Nothing on origin is replaced, so this is bounded, but the card's own bound does not say so.
Card 2 (checkout) and card 4 (add) both delete a branch the resolver created when a later step fails, so card 3 is also the odd one out.
**Fix:** Pick one and state it in the Intent.
Preferred: when the resolver created the branch (`fromOrigin`) and `adoptWeftWorktree` then fails, delete it through `deleteBranch` with `ownedManagedBranch`, as `rollbackSwitch` does, and add a row to card 3's tests.
Alternative: keep the branch and add one sentence naming the live-pair consequence and the bound, that nothing on origin changes and the hub can remove the pair again.

### [LOW] Card 2 documents no checkout behavior change

**Location:** `_lyx/plan/02-weft-branch-resolver-checkout.md` (Edit list and Docs, absent).
**Issue:** The decision record scopes docs to the touched packages' `doc.go` for changed observable behavior, and CLAUDE.md puts them in the same commit.
Card 2 makes `checkout` adopt an origin weft, with or without an archive tag, and makes an unreachable origin stop the verb where it used to fork.
Card 3 documents reconcile and card 4 documents add, so checkout's change is documented nowhere.
**Fix:** Add `internal/fabricengine/doc.go` to card 2's Edit list.
Add a short Docs paragraph that `Checkout` resolves the weft branch local, then origin, then fork, that an unreachable origin stops it, and that a weft repo with no `origin` remote forks as before.

### [LOW] The add help text still says the weft branch is always forked

**Location:** `_lyx/plan/04-add-adopts-live-pair.md:65` (Docs); `internal/fabriccli/fabric.go:172`.
**Issue:** The `lyx fabric add` `Long` text says "The new weft branch is forked from the HEAD of the worktree you run `lyx fabric add` from".
For a live pair, card 4 adopts the local or origin weft branch and forks nothing.
The card's Docs line covers `doc.go` only.
**Fix:** Add `internal/fabriccli/fabric.go` to card 4's Edit list.
Add one sentence to the `add` `Long` text that a pair with an existing weft branch, local or on origin, is adopted rather than forked.

### [LOW] A nil-seams row repeats every existing Finalize test

**Location:** `_lyx/plan/06-finalize-config-notice.md:50`
**Issue:** `newTestDeps` in `internal/landingshed/publish_test.go` leaves every optional seam nil.
So each existing Finalize test, for example `TestFinalize_MergeMarkPushOrderAndVerdict`, already asserts that Finalize with nil `ConfigChanges` and `Notify` behaves as before.
The row "nil seams change nothing" covers a behavior no new test needs to pin (`PATTERN-test-economy`).
The case that is not covered is a non-nil `ConfigChanges` with a nil `Notify`, which has to do nothing.
**Fix:** Replace the row with "a non-empty diff and a nil `Notify` still reaches Done and queues nothing", or drop it.

## Focus verdicts

**Test economy, card 1 `ResolveParent` lock-directory error.**
Not a defect.
The card plans the error assertion once, as the "before reconcile" assertion of step (b) in `internal/hubgeom/geometry_integration_test.go`, and plans no `parent_test.go` case.
The existing `parent_test.go` has no lock-directory-missing case, and its `hubWithOrigin` helper creates `.weft` by hand.
A second `parent_test.go` case would assert the same error through the same public surface and duplicate step (b).
Step (a) and the existing fabricengine reconcile recreate test (`TestReconcile_RecreatesHandDeletedWeftWorktree`) differ in surface and assertions: the first goes through the CLI and asserts `.weft/` and the resolved parent, the second asserts only the action and branch.

**Test economy, card 4 origin-holding-only-the-weft adopt.**
Not a defect.
The card pins it once, in the converted `add_remoteleftover_integration_test.go` row, and plans no CLI row for it.
The CLI row builds a pair from both origin branches, so it is the D8 wiring test.
The two rows share the "weft tracks origin" assertion, but the fabricengine row's own assertion, that the warp forks from HEAD, is not in the CLI row.
The CLI row's last assertion is wrong, see F1.

**Test economy, card 6 fake row versus real-landing step.**
Not a defect beyond F5.
The fake row pins the notice composition (task branch, each file, both SHAs, the re-apply instruction), which the integration step does not assert.
The integration step pins the resume re-queue over an already-landed parent and the parent's byte-identical config, which no fake row can reach.

**Attack surface, card 3 adopt failure after the resolver created the branch.**
The Intent states what remains and that origin is untouched, but not the consequence for card 4's live-pair rule.
That gap is F2.
