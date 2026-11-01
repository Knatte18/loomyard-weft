```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checked each refusal, error, next step, row and test the fix added for whether it can be carried out from the state it names and leads to progress.

Ways forward the fix introduced, as I read them:
1. D3: "It refuses to write a target whose file is not on fabric's exclude list (D5)."
2. D3: `lym heddle render` has fabric seed the lines first through `fabricengine.SeedWarpExclude` (new); fabric seeds them before either D6 event.
3. D3: the per-worktree render lock under the pair's `.lym` tree, its path handed to heddle with the warp root.
4. D3: removing a stale marked render is never refused by the exclude-list check.
5. D6: a detached seam failure is log-only and names the worktree and `lym heddle render`; the stale file stays until the next take-in or a hand render.
6. D7: the origin owner read with no run ledger registered fails as `ErrBoardReplayMissing` does, naming the registration.
7. D7: set-aside tag `set-aside/replayer/<name>/<short sha>`.
8. Open risks: worktrees from before the landing recover through `lym heddle render`, which seeds first, or a take-in, before which fabric seeds.
9. D3 and D6 acceptance cases added by the fix.

Findings:

- LOW — D3: "It refuses to write a target whose file is not on fabric's exclude list (D5)."
  The fix rewrote this refusal as its own bullet, but it still names no way forward, while D2 holds every heddle refusal to name one (PATTERN-refusal-way-forward).
  Evidence: D2 says "Refusals: … each naming the way forward". Since `fabricengine.SeedWarpExclude` now seeds whatever `warp_exclude` lists, the refusal is only reachable when the operator has taken the path off `warp_exclude`. The step that makes progress is putting the path back on fabric's `warp_exclude` key, and the bullet never says so.

- LOW, outside my lens and noted only: the D7 acceptance criterion names the tag `set-aside/replayer/heddle/<short sha>`, but D7 never names heddle's replayer `heddle`. It only says "heddle registers one for `heddle.json`".

No finding for items 2 to 8. Each step can be reached from the state it names:
- `SnapshotTagged` (`internal/fabricengine/snapshotuncommitted.go:113`) prefix-matches `set-aside/<slug>/`, but only in the weft repository and only annotated tags. Replay set-aside tags are lightweight, so a run slug `replayer` does not count heddle's set-asides.
- The registration that the owner-read failure names is the root pre-run's, and D7 already states it.

Tool calls:
- grep for `setAsideTagPrefix` readers and for a no-op check on PATTERN `changes` bullets: my context held neither who parses set-aside tags nor whether an unchanged `changes` text is refused.
- `sed` of `snapshotuncommitted.go:100-125` and a grep for fabric's reserved names: to see whether the set-aside prefix reader would take a replayer tag as a run's.
