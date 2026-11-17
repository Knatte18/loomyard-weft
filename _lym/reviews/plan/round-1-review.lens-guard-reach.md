```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal and branch the cards add, check that a real entry point can reach it, and find guarded sites that recur where no card guards them.

**Per guard, reachable or not**
- Card 1, whole-commit refusal over another machine's new hold: reachable. A rejected `WriteAdopt` push is reached through `BeforePush`, and card 2's `TwoClones…` row reaches it.
- Card 1, replace or remove of a hold matched in origin's list: reachable. Rows exist.
- Card 1, `Lost` for a parent hold origin no longer holds: reachable, but **no named test row** (see F4).
- Card 1, registry `Lost`: reachable by planting. A row exists.
- Card 2, cases 3, 4 and 5 (own hold over another set; another machine's hold; take-hold over another set): reachable from the CLI. Each has a row.
- Card 2, registration refusal: reachable. Row exists.
- Card 2, post-push re-read refusal naming the winner: reachable through `BeforePush`. Row exists.
- Card 2, `FixHold` with no hold: reachable only through a concurrent green check clearing the hold. Unit row exists.
- Card 2, `FixHold` over another machine's hold: reachable after a `--take-hold` during the check. Row exists, but the same state recurs unguarded on other paths (F1, F2).
- Card 2, `ClearHold` with another machine's hold, reported false: reachable. Its caller does not act on false (F1).
- Card 3, exhausted retry: reachable through `takeoverBeforePushHook`. Row exists.
- Card 3, anchor-change refusal through `DropLocal`: reachable. Row exists.
- Card 3, retry's reset onto origin's new tip: reachable. The local-tip archive guard does not cover the parent's records branch (F3).
- Card 4, shell-guard deny and allow rows: reachable.
- Card 5, `UnderAdopt` true branch: the state cannot occur in production (F5).
- Card 6, undecodable board joining the failures: reachable. Row exists.
- Card 7, propagated read errors: reachable.
- Card 8, `no_hold`: reachable (a fix task landing after the hold cleared). Row exists.
- Card 8, a first subject outside the hold: reachable through a stale fix task's landing under a newer hold. Row exists, but it exits non-zero, which `no_hold` was made to avoid (F6).
- Card 8, lock-held check, failed `ClearHold`, `BeamUp` or `DropBoltHold`, the limit, an open own fix task: all reachable, all have rows.
- Card 9, take-hold with `--all`, take-hold with `--archives`: reachable. Rows exist.
- Card 9, `holdStands` finding the hold gone or naming another machine: reachable before steps 2 to 7. Rows exist.
- Card 9, `TakeOver` error, catch-up error, workspace warning, anchor change: reachable. Rows exist.
- Card 9, `movedRun` branches: reachable. Rows exist.

**Findings**

F1 — BLOCKING, decision, cards 8 and 9 (`runHoldCheck` step 3 and step 4, `runForeground` steps 7 and 8). A hold taken over during the check is caught only on the red `FixHold` path. If another machine runs `--take-hold` while the gate runs, the most likely window:
- On green, `ClearHold` reports false and returns no error, yet the call goes on to `BeamUp` and `DropBoltHold`.
- On red, card 8 step 4 runs `BeamUp` per subject after the filing even when `FixHold` refused.
- Card 9 runs `runBeamSteps` "on green and on red alike". Its last hold re-read is before the check ("no re-read follows them").

So the old adopter starts dobbies. This breaks D6 ("re-reads its own hold … before each step from step 2 on … starting nothing") and the acceptance criterion "stops at its next hold re-read, naming that machine, and starts no dobby". Evidence: `_lym/plan/08-adopt-check-ends-on-hold.md:70-73`, `_lym/plan/09-adopt-foreground.md:185-186,199`, `_lym/plan/02-board-adopt-write.md:53-54`; decision record lines 63 and 153.

F2 — MEDIUM, design, card 2 (`FixHold`, `ClearHold`). The other-machine refusal is judged only over the pulled store, with no re-read after the push.
- On a rejected push, card 1's replay refuses the whole commit or reports the hold `Lost`.
- The Bolt replay only logs a `Refused` finding (`internal/fabricengine/bolt.go:393`), and `pushOwnerCommit` then reads "not ahead" and returns nil (`internal/boardengine/ownerset.go:356-376`).
- So `FixHold` and `ClearHold` report success for a change origin refused.

`WriteAdopt` re-reads after its push (card 2 line 47); its two sibling setters do not. Evidence: `_lym/plan/02-board-adopt-write.md:51-55`, `_lym/plan/01-board-replay-holds.md:27-32`.

F3 — MEDIUM, design, card 3 (`TakeOverRun` steps 2, 4 and 5).
- Step 2's archive of local tips origin lacks, and step 3's `DropLocal`, cover the moved pair's branches.
- Step 4 also commits onto the parent's records branch, the prime's for a root child, and step 5's retry resets "the records branch" with `reset --keep` to origin's new tip.
- The card names no archive of this machine's unpushed commits of other runs on that shared parent branch before such a reset.
- Card 9 runs the catch-up that pushes owed commits only at step 4, after the takeover, so owed transitions after an absence can be dropped.

Evidence: `_lym/plan/03-fabric-takeover-writer.md:38-45`; `internal/fabricengine/destroy.go:1977-1990` (DropLocal touches only `target.slug == slug`, not the prime).

F4 — LOW, consistency, card 1 (`FabricReplay.Apply`). The branch "a parent hold origin no longer holds is reported `Lost` and its change is not applied" has no `TestFabricReplay` row. Every other hold branch has one. Evidence: `_lym/plan/01-board-replay-holds.md:32,42-48`.

F5 — LOW, design, card 5 (`UnderOwnHold`, `Deps.UnderAdopt`, `OwnerWatch.UnderAdopt`). The guarded state cannot occur in production: "the subject's owner reads as another machine while this machine's hold lists the subject".
- Card 2 writes the hold and the subjects' owner in one commit.
- A take-hold rewrites both to the other machine.
- A nested beam keeping its own foreign owner is excluded by the card itself.

So the dobby's "stay idle under the hold" branch and the archive skip in `ownerArchiveBeforeRefusal` never fire. The tests plant a hold over root only to exercise the accessor. Evidence: `_lym/plan/05-own-hold-reader.md:36-44`, `_lym/plan/02-board-adopt-write.md:39-45`, `internal/dobbyengine/handover.go:39-57`.

F6 — LOW, design, card 8 (`checkCmd`, `runHoldCheck` step 1). Card 8 makes `no_hold` exit 0 so that `fronttask`'s `adopt-check` row does not retry at every settle. The other end a stale fix task's landing reaches, "a `first` outside the hold's subjects", is a refusal that exits non-zero. That case recurs, retried at every settle with nothing to change it. Evidence: `_lym/plan/08-adopt-check-ends-on-hold.md:67-68,87`; `internal/fronttask/fronttask.go:33-36`.

Tool calls:
1. `grep Refused` in `internal/fabricengine/boltreplay.go` and `bolt.go`: to learn how a Bolt push treats a replayer's refused findings; the context had the board replayer but not fabric's push handling.
2. `sed bolt.go:360-420`: to confirm refused findings are only logged, not returned; needed for F2 and the card 1 and 2 reach paths.
