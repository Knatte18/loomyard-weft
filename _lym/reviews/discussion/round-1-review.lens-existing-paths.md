```yaml
lens: existing-paths
outcome: reported
```

I traced the paths in the existing code that each Decision's state meets (existing-paths lens).

**Paths traced, per decision**
- D1, beam.yaml `pull_request`: config load, template reconcile and `hubreconcile` add a template key the usual way. Survives.
- D2, board `pull_request` field: the run lock refuses a held entry's change to any field but status (`internal/boardengine/doc.go:203`), so the field stays fixed during the run. Survives.
- D3, retired `loom.pr_review` and `darn.pr_review` keys and the dropped `pr_review_setting` row key:
  - Recipes are read only as embedded in the binary (`internal/shedrecipe/doc.go:98`), so no hub copy of a recipe still carries `pr_review_setting`.
  - The readers that remain are `internal/loomengine/config.go:422,552`, `internal/darnengine/config.go:61,99`, `internal/darnengine/doc.go:3` and `contracts/recipes/*:536-541/222-227`. All of them are retired together with the keys.
  - Survives.
- D4, `ParseOwnerRepo` gains URL shapes. Its only other use is the selfreport path, and that is helped, not harmed. Survives.
- D5, a cache file per account: `cache.go` with `invalidateCachedToken` on a 401 has a single file today. D5 states its own file per account and keeps the 401 replay. Survives.
- D6, record in the run directory: `shedrun.RunDir` is the durable, fabric-synced `_lym/shed/&lt;slug&gt;` under the task worktree's anchor (`internal/shedrun/paths.go:120`). Survives while the worktree is present.
- D7, session started by a new start with a prompt modelled on `Runner.Open`:
  - Stopping it through `Runner.StopStrand(guid)` (`internal/shuttleengine/stop.go:51`) survives if the strand GUID is kept somewhere.
  - `StopStepRuns` (`stepstop.go:43`) reaches only runs that have a run directory, so a step stop never reaches this session. D7 stops it itself, so that holds.
  - "Resumes it by its session id" breaks (finding 3).
- D8, the `pr-review` gate through verifytree:
  - The verify marker is live during the gate, so the quiet check is held off for that span (`internal/battencli/wire.go:619-625`). Survives.
  - The gate's verify mark replaces the `PR #&lt;n&gt;` mark, which D10 already handles. Survives.
- D9, a long in-process wait that holds the run lock:
  - `EndForCancel` ends the owned processes, kills the loop, then waits for the run lock (`internal/pairteardown/cancel.go:53-125`). Cancel reaches the waiting row. Survives.
  - Pause is handled by the D9 transient mark. Survives.
- D10, a wait marker the quiet check already reads: `readAgentActivity` (`internal/battencli/wire.go:595-645`) breaks (finding 2).
- D11, Finalize merges on GitHub, then fast-forwards the local parent pair:
  - Fabric's two-side merge, as `internal/landingshed/finalize.go:10-11` describes it, breaks (finding 1).
  - The already-landed path for a pull request merged on GitHub (`finalize.go:164-166`) survives.
- D12, pair teardown closes the pull request:
  - Cancel, `lym fabric remove` and batten's Worktree-Teardown with the task worktree present survive.
  - A teardown with the task worktree gone (`cancel.go:56`, `TaskWorktreeGone`) breaks (finding 4).
- D13, recipe rows: recipes are embedded and the header comments are retired with the rows. Survives.

**Findings**

1. BLOCKING · design · D11, the "Otherwise" bullet. A GitHub merge in place of the local parent-side merge lands only the warp side.
   - The record side (weft) of the task branch is never in the pull request (`internal/landingshed/finalize.go:10-11`): "the remote service only ever sees one of the two sides, so the other side's branch was never in the pull request at all, and after a remote-side merge the visible side reports already-up-to-date while the other genuinely merges".
   - D11 then fetches the parent and fast-forwards the local parent pair to its upstream. That fast-forward never merges the task's weft branch into the parent's weft.
   - It also never records the `Warp-SHA` correspondence that `Fabric.Merge` (`internal/fabricengine/merge.go:364`) writes for the new warp commit GitHub created.
   - D11 should hand the run to Finalize's existing already-landed path after the GitHub merge. That path is the one that merges the other side. D11 is silent on this.

2. BLOCKING · design · D10. No marker the quiet check already reads fits a row that waits on GitHub.
   - `readAgentActivity` (`internal/battencli/wire.go:589-645`) counts exactly four kinds as a live wait. None of them fits this row:
     - **A verify marker** is owned by verifytree and live only while a gate runs.
     - **A parent-lock wait record** contradicts D9's "takes no parent lock at any point".
     - **A child at Beam-Await** is keyed on the row name `pawlengine.NameBeamAwait` (`wire.go:652-665`).
     - **A shuttle wait marker that is not held** is written only by `Run.Wait`, for a gate entry, background shells or a held turn end (`internal/shuttleengine/waitmark.go:19-27,246`). The D7 session is started un-awaited and has no `Run`, so it has none.
   - As written, the quiet notice fires on every PR wait longer than `notice_quiet_min`. That breaks the operator's creel #20 decision.
   - The record has to name the mechanism. For example, the readers could learn a child running at `PR-Review`, beside Beam-Await. That is a change to `battencli`, not a marker the check "already reads".

3. MEDIUM · design · D7. "Finds the run's session strand live", "stops it through shuttle's stop verb" and "resumes it by its session id" read state that no stated file holds.
   - `Runner.Open` writes no run directory and no run record (`internal/shuttleengine/open.go:1,38-45`). It returns the strand GUID and the session ID only to its caller (`open.go:100`).
   - The D6 record holds number, URL, base and gated head only.
   - `StopStrand` needs the GUID (`stop.go:51-54`).
   - No `Runner` method resumes an opened session by its ID. D7 names only the new start with a prompt as a shuttle gain.
   - The resume that D9's approval path and D7's re-entry rest on has no stated home or shuttle capability.

4. LOW · design · D12, "that covers … every other teardown caller". A teardown with the task worktree already gone cannot read the D6 record, so the pull request stays open.
   - `EndForCancel` and the teardown sequence run "for a present or a gone task worktree" (`internal/pairteardown/cancel.go:43,56`).
   - The record lives under the task worktree's anchor in `shedrun.RunDir` (`internal/shedrun/paths.go:120-121`).
   - With the worktree gone the record is unreadable.

**Tool calls** (5 of the 6 allowed)
- grep of `pr_review` across the recipes, config and code, plus `interruptpolicy.go`: to find every reader of the retired keys and the PR-Review interrupt policy.
- grep of shuttle open and resume and the pairteardown functions: to see whether shuttle can resume an opened session and how teardown is ordered.
- the head of `pairteardown/doc.go` and where recipes load from: to learn the teardown sequence and whether hub copies of recipes exist.
- `EndForCancel` and the `shedrun` path functions: to see whether cancel reaches a row holding the run lock, and where the D6 record lives.
- shuttle's stop and resume method signatures: to see which state the D7 stop and resume need.
