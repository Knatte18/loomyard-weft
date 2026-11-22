```yaml
lens: existing-paths
outcome: reported
```

**Lens existing-paths: findings on the code paths that already read or write the state each decision touches.**

**Findings**

- **MEDIUM, D1.** The record says "A nested beam is governed by its own `beam.yaml` stack, never by its parent's". In practice that stack usually starts with the parent's value.
  - Evidence: `beam.yaml`'s worktree layer is `ConfigFile(Anchor)`, a file tracked on the worktree's weft branch. `ReadConfigChanges` (`internal/fabricengine/configchanges.go`) compares that file "since its fork point". So a beam or task branch forked from an autonomous beam inherits `autonomy: true`.
  - Effect: a nested beam made under an autonomous beam starts autonomous unless someone writes `false`. The record does not mention this inheritance.
- **MEDIUM, D1 with Finalize.** When an autonomous beam lands, Finalize's config-change notice tells the orch to re-apply the beam's `autonomy` change on the parent.
  - Evidence: `configChangeNotice` in `internal/landingshed/finalize.go` lists changed per-worktree keys and says "landing does not carry them to the parent branch, so re-apply each change meant for the parent on the parent's copy". Under D5 the orch acts on such lines alone.
  - Effect: the record does not say `autonomy` is never re-applied upward. An autonomous orch could turn the prime, and all its root tasks, autonomous without the operator.
- **MEDIUM, D6.** "Posts once per drain" does not say where the once-only state is kept, and Beam-Await is re-entered on restart.
  - Evidence: `loomshed/doc.go` names Beam-Await's interrupt policy as reinvoke. `BeamAwait.Call` rebuilds `awaitRun` on every entry. A driver revive, `lym beam start` or a hibernate wake re-enters the row.
  - Effect: a flag held in memory posts `[beam drained]` again after each re-entry. The orch would then redo D7's steps, possibly a second `describe` or `pr open`. The state must be stored, or the repeat made harmless.
- **MEDIUM, D3.** `discussion_interactive` is read twice in the same place, and both reads must be forced.
  - Evidence: `internal/loomengine/discussiontable.go` reads it for `Interactive:` and again for `discussionModeRules(!cfg.DiscussionInteractive, …)`. `judge_interactive` is likewise copied into the resolved config (`internal/driftengine/config.go`) and read in `internal/driftengine/judge.go` twice: the branch and `Interactive:`.
  - Effect: forcing "where each key is read" fails if one read is missed. The session would run autonomous with operator-mode prompt text, or the reverse. The safe point is the resolved `cfg` handed to the table or judge, not each use.
- **LOW, D4.** Flipping the PR-Review spec to `Interactive` false brings in autonomous-session paths the record does not mention.
  - Evidence: `shuttleengine/hold.go` `notifyHeld` sends a hold notice to the parent on each held turn end. The tackler prompt reads `Spec.Interactive`.
  - Effect: the PR-Review session waits on parent mail across many turns, so the orch gets a hold notice per wait on top of the `[pr-review request]`. This survives but adds load. A PR-Review session already live when autonomy flips keeps its old mode until it respawns.
- **LOW, D4.** The PR-Review prompt has no mode marker today; it is filled with `parentdirective` and edit-directive markers only (`RenderPRReviewPrompt`, `prsession.go`). The record's Review criterion needs the stencil to know about autonomy, so the render needs a new input. Plan can settle this, but nothing in the record names it.
- **LOW, D8 with D9.** `contracts/orchrole/role-slack.md` says "lym posts the final reply of each of your turns to your channel, so you need not track the presence". Under D8 that is false. D9 adds exceptions only to `orch-role.md`, so `role-slack.md` would keep contradicting D8.
- **LOW, D7, not verified.** Step 1's `lym beam merge-parent` files a merge task. If that task is filed draft, D6 counts it as open, and the record says releasing drafts stays the operator's. Then D7 stalls at step 1 until the operator releases it. The `loomshed` doc says "Every filed task is draft" for pawl-filed tasks. I did not check merge-parent's task status.

**Paths traced**

- D1, Seed-Child `writeRunSettings` with the local layer refusing a Guard key: survives.
- D1, `lym config beam --set` in a member worktree: harmless, since members read their parent.
- D1, prime worktree file on main, inherited by tasks forked from main: harmless.
- D1, fork inheritance of a beam's worktree file: breaks the nested-beam claim.
- D1, Finalize `configChangeNotice` re-apply advice: risky under D5.
- D2, `parentPullRequestMode` pattern, root task's parent being the prime: survives.
- D3, `discussiontable.go` (two reads) and `judge.go` (two reads): survives only if forced at the resolved config.
- D3, `landingdeps.go` `ParentReviewWanted`: survives.
- D4, `sessionSpec` rebuilt per spawn: survives.
- D4, `OperatorWaitRows` keeping PR-Review a live wait for dobby and the watchdog: survives.
- D4, `askParentReview` asking once per record: survives, since the stencil re-asks after fixes.
- D4, hold notices: extra load.
- D6, Beam-Await reinvoke and restart: breaks without stored state.
- D6, `Notify` seam already used for blockers: survives.
- D8, `replyHook` reads the presence at call time, and `promptHook` is unaffected: survives.
- D8, `role-slack.md` wording: contradicts.
- D7 and Finalize merge path: unchanged, survives.

**Tool calls**

1. Grepped readers of `DiscussionInteractive`, `JudgeInteractive` and `PRReviewAskParent`, plus `ConfigFile` and `AnchorPath`: my context had no reader list for the keys D3 forces.
2. Grepped `ConfigFileRel` users and `WorktreePath`: to learn whether the worktree layer is a tracked file that moves with branches.
3. Read `landingdeps.go` around `perWorktreeConfigRels`: to see what landing does with per-worktree config files.
4. Grepped `perWorktreeConfigRels` callers: to find the Finalize seam that uses them.
5. Grepped `ConfigChanges` in landingshed and `ReadConfigChanges`: to confirm fork-point comparison on the weft branch.
6. Read `finalize.go` `configChangeNotice`: to see whether landing carries the file or tells the orch to re-apply it.
