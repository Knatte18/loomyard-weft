---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-weft-lock-dir.md:47
    summary: Card 1's add rows cannot fail without D1's Add change, because WriteOrigin already creates the lock directory on both add paths; the one add path that depends on D1 (an adopted weft that already carries a record) and the dormant raw fork are never asserted to have the lock directory
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/02-weft-branch-resolver-checkout.md:33
    summary: A failing weft switch right after the resolver created the local branch from origin leaves that branch behind, because Checkout's error path passes no created branch to rollbackSwitch
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/04-add-adopts-live-pair.md:67
    summary: The rollback rows are planned in add_remoteleftover_integration_test.go, whose scope is pre-flight probes, with the failure injection left open, while add_rollback_adopt_test.go holds the existing Add-rollback tests for adopted branches and is not a card target
---

### [MEDIUM] Card 1's add rows are vacuous, and the paths D1 changes are left unasserted

**Location:** `_lyx/plan/01-weft-lock-dir.md` (Tests, `weftlockdir_integration_test.go`), `_lyx/plan/04-add-adopts-live-pair.md` (Tests, `origin_integration_test.go` and `adoptremote_integration_test.go` bullets), `_lyx/plan/03-reconcile-adopts-origin-weft.md` (Tests, last bullet).

**Issue:**
`Topology.Add` step 10c calls `WriteOrigin`, and `WriteOrigin` goes through `originLockPath` to `ensureWeftLockDirAt`, so today every Add that writes the record already leaves `.weft/` in the new weft worktree, on the fork path and on the local-adopt path alike.
Card 1's new file `internal/fabriccli/weftlockdir_integration_test.go` asserts exactly that for both paths, so its rows pass before card 1's Add edits and pin nothing D1 changes.
The one Add path where D1's adopt-path `ensureWeftLockDirAt` is the only thing creating `.weft/` is an adopted weft that already carries an origin record, because card 4 then skips `WriteOrigin` and the record commit.
Card 4's rows for that path (`origin_integration_test.go`: record kept, no new record commit) do not assert `.weft/`, and its `adoptremote_integration_test.go` origin-adopt row asserts `.weft/` on a weft with no record, where `WriteOrigin` creates it anyway.
D1's other direct path, `createWeftWorktree` through reconcile's dormant `raw_adopted` fork, has no `.weft/` assertion either: card 3's last bullet (origin remote removed, raw warp worktree still forks dormant) asserts the action only.
Net effect: dropping the `ensureWeftLockDirAt` call from Add's adopt branch, or from `createWeftWorktree`, leaves every planned test green, so the stacked-task move D6 exists for could silently regress into the original bug.
The `.weft/` assertion for the local-adopt row also repeats card 4's origin-adopt `.weft/` assertion, which goes through the same `containedWorktreeAdd` adopt call.

**Fix:**
Drop `internal/fabriccli/weftlockdir_integration_test.go` from card 1 and its Tests bullet, with the matching sentence in the Intent.
Assert `.weft/` in the rows that can discriminate:
(1) card 4's `origin_integration_test.go` row for an adopted weft that already carries an origin record: `.weft/` exists in the new weft worktree, in addition to the kept parent and the absent record commit;
(2) card 3's no-origin raw-warp row: `.weft/` exists in the dormant weft worktree;
(3) keep one fork-path assertion, as an added assertion in card 4's `adoptremote_integration_test.go` origin-weft-only row (the fork warp path with an adopted weft) or in an existing CLI add test, naming which, so the acceptance criterion "on both the fork and adopt paths" stays covered without a new file.
Card 1's Tests then list only the `parent_test.go` row and the `reedgeom_integration_test.go` reconcile rows, which do fail before the fix.

### [LOW] A failed weft switch after an origin-created branch leaves the branch behind

**Location:** `_lyx/plan/02-weft-branch-resolver-checkout.md` (Intent, `switchOrForkWeft` paragraph).

**Issue:**
D7 and card 2 say an origin-tracked branch is treated as a created branch on rollback.
The card wires that only for a failure after `switchOrForkWeft` returns `created=true`.
When the resolver has created the local `<b>-weft` branch and `git switch` then fails (for example the weft worktree has local changes that conflict), `switchOrForkWeft` returns an error, `Checkout` calls `rollbackSwitch(..., "")`, and the new local branch is never deleted.
Nothing is lost and the next checkout adopts it as a local branch, but it contradicts the stated rule that the created branch is removed on failure, and the planned tests reach only the junction-failure case.

**Fix:**
State in card 2's Intent that a switch failure after the resolver created the branch deletes that branch through `deleteBranch` before returning (or that `switchOrForkWeft` returns `created=true` together with the error and `Checkout` passes it on), and extend the `checkout_rollback_test.go` row's failure injection or add one assertion so the created-then-switch-failed case is covered.

### [LOW] Card 4's rollback rows sit in the pre-flight test file and leave the injection open

**Location:** `_lyx/plan/04-add-adopts-live-pair.md` (Tests, `add_remoteleftover_integration_test.go` bullet).

**Issue:**
Card 4 plans three rollback behaviors in `add_remoteleftover_integration_test.go`: a fast-forwarded local weft branch stays at origin's tip after a later failure, a resolver-created origin-tracked weft branch is deleted, and origin branches are unchanged.
That file's header scopes it to Add's pre-flight probes.
The existing tests for Add's rollback of adopted and created weft branches are `TestAddRollback_AdoptedWeftBranchSurvives` and its siblings in `internal/fabricengine/add_rollback_adopt_test.go`, which inject a deterministic post-adopt failure with a blocker file at the portal.
The card does not target that file and names the injection only as "(rollback hook or injected failure)", which leaves the implementer to invent a second mechanism beside the existing one.

**Fix:**
Move the three rollback rows to `internal/fabricengine/add_rollback_adopt_test.go`, add it to card 4's Edit targets, and name the existing portal-blocker injection as the failure source.
The pre-flight rows (adopt-only origin weft, fast-forward accepted, true divergence refused, archive-tagged weft with a local branch, skip options) stay in `add_remoteleftover_integration_test.go`.

## Focus entries settled without a finding

- **D1, same `containedWorktreeAdd` call:** card 4 adopts a resolver-created branch "the same way (`containedWorktreeAdd` without `-b`)" in the adopt branch card 1 edits, and says card 1's adopt-path `ensureWeftLockDirAt` still creates `.weft/`; no adopt path is left without it.
  The paths are carried; what is missing is a test that would notice (F1).
- **D6 and D7 bullets:** every D6 bullet (archive tags consulted only without a local weft, `SkipGit`/`SkipPush`, fast-forward, record preservation, rollback scope, retired refusal) and every D7 bullet (resolver before the raw and unmanaged branches, archive-guard bypass, per-pair `Error`, checkout rollback) is carried by cards 3 and 4, and no card step goes beyond what D5 to D7 license.
- **Granularity:** card 1 bundles D1 to D3, which are one bug and one way forward (the D3 error names the reconcile heal), and card 4 is D6, whose live decision, probes, adoption and rollback are coupled through the one `live` answer.
  Neither bundles parts that review separately; not a defect.
- **Test economy:** besides F1, no planned file or row repeats an existing test.
  The `parent_test.go` row and the `reedgeom_integration_test.go` reconcile rows differ in tier and in what they pin (message wording versus heal through the real verb), and the raw and managed rows of card 3 differ in the wiring repair they reach.
- **Attack-surface question:** cards 2, 3, 4 and 6 each state a bound in `**Intent:**`, and card 1 bounds reconcile's new write to a pair whose weft worktree exists and lacks `.weft/`, with no action change; card 4's fast-forward is bounded by git's own refusal of a non-fast-forward.
- **Verify coverage:** `## verify:` runs the untagged, `integration` and `tmux` tiers for all five packages the cards target, and compiles the `llm` tier; not a defect.
- **Sequencing:** cards 2 before 3 and 4 (resolver), 1 before 4 (`Topology.Add`) and 5 before 6 (`ReadConfigChanges`) hold, and each card builds on the ones before it; the `landingdeps_test.go` drift guard needs no change because card 6 fills both new fields.
