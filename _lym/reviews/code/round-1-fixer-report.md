# Round 1 fixer-report

The review's `decision`-class finding F3 went to the parent, who recorded its decision (creel #638); it is not part of this report.
The gate (`lym gate test`) passed at tier 0 and tier 1 over `internal/fabricengine`, `internal/battenshed` and `internal/battencli` after the first round of fixes, and at tier 0 after the self-check follow-ups, which touched comments and one test file header only.

## Changes

- **F1** — fixed as proposed. `TestAdd_HoldsTheCallersLock` now records whether the pair's worktree path exists when `Hold` is called (asserted absent) and when its release runs (asserted present). A follow-up updated the file header to the pinned order. Commits `fabricengine: fix F1`.
- **F2** — fixed as proposed. `countingHold` in `run_test.go` replaces `holdNothing` in the three named places: the lifecycle test asserts one take and release for the first create and no further take over the complete pair; the held-member rows assert two takes and releases (`late`) and one (`crashed`); `TestRefuseHeldMember` gains a held-beam row whose hold fails with a sentinel, asserting `errors.Is` and the take count. The "no pair removal ran" half is held by the sentinel check alone: a removal started before the hold would fail with a different error. Commit `battencli: fix F2`.
- **F4** — fixed differently from the proposal (see Self-check). The proposed wording, "never around an origin probe", is false: `Add` calls `probes.Wait()` under the hold, so the hold covers whatever probe time remains after the pre-flight reads. The first round carried out the proposal, and the follow-up replaced it. The claim now reads that the probes start before the hold and overlap its wait, that `Add` waits for them under it, and that the adopt fetch, the weft fast-forward fetch and the archived-leftover replace also run under it. Sites: `HoldPrimeLock` in `create.go`, the Worktree-Create paragraph of `battenshed/doc.go`, `AddOptions.Hold` and the probe comment in `fabricengine/add.go`, the comment above `CreateWorktree` in `battencli/wire.go`, and the Worktree-Create line of `docs/overview.md`. Commits `battenshed: fix F4` (two).
- **F5** — fixed as proposed. `AddOptions.Hold` in `add.go` states that `Add` calls `Hold` under the pair's teardown lock, and why that is safe. Commit `fabricengine: fix F5`.
- **F6** — fixed as proposed, then widened by the self-check. The `primelockwait.go` header now says every producer that holds the prime lock calls `acquirePrimeLock` in place of a single `Acquire`, since the Land row calls it too; the opening sentence of the prime-lock paragraph in `battenshed/doc.go` reads the same way. Commits `battenshed: fix F6` (two).
- **F7** — fixed as proposed. `HeldAgainAfterRelease` in `create_test.go` takes and releases two holds in one `Call` and asserts `Done`, two `Acquire` calls and two releases. Commit `battenshed: fix F7`.

## Gate repair

The gate failed `TestEnforcement_FabricVocabulary/tree-scan`: the F4 wording in `internal/battenshed/create.go` and `internal/battenshed/doc.go` named the weft branch, and `battenshed` is outside the vocabulary's owner set.
Both now say "a behind branch" of the pair, with no weft or warp token.

## Self-check

- fix-proposal, MEDIUM — F4's reworded claim "never around an origin probe" is false, since `Add` waits for the probes under the hold. Fixed with my own change, described under F4.
- fix-proposal, LOW — the `create.go` fetch list was presented as the only network steps under the hold. Fixed with F4's rewrite, which names the probe wait too.
- fix-code-correctness, LOW — the `wire.go` comment claimed a create waiting on origin holds up no other. Fixed with F4's rewrite.
- fix-consistency, LOW — the `primelockwait.go` header omits the Land caller. Fixed as proposed, and the `doc.go` paragraph opening was widened the same way.
- fix-affected-set, LOW — same finding as the fix-consistency one (`land.go` calls `acquirePrimeLock`, and `doc.go` named two producers). Merged with it and fixed.
- fix-consistency, LOW — the `add_hold_test.go` file header no longer described the test. Fixed as proposed.

All four forks returned a report; each is saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.

## For the parent

`Add` still waits for the origin probes under the prime lock (`probes.Wait()` before the first mutation, with the hold released at return). On a slow or degraded origin one create can hold up another for that wait, which the Goal's "side by side" idea aimed to remove. The comments and `docs/overview.md` now say so; narrowing the code further would be a change to `Add`'s ordering and is not made here.
