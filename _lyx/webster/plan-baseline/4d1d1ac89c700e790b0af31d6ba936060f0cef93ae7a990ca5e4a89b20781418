# Card 26 — name-beam-start-everywhere

**Create:**
- `cmd/lyx#TestRemovedVerbs_NamedNowhere`

**Edit:**
- `internal/orchengine/launch.go`
- `internal/hibernateengine#reconcileAtWake`
- `internal/hibernateengine#holdRuns`
- `internal/hibernateengine#TestWatch`
- `internal/hubreconcile#reconcileWayForward`
- `internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward`
- `internal/fabricengine#BranchOutcome.anchorStopWayForward`
- `internal/fabricengine_test#TestReconcileOrigin_Moves`
- `internal/hibernateengine/doc.go`
- `internal/fabricengine/doc.go`
- `internal/battencli/cli.go`
- `pattern/PATTERN-pane-text-injection.md`
- `plugins/ly/skills/board/SKILL.md`
- `docs/overview.md`

**Intent:**
D4's last rule: every way forward, notice, doc and skill naming `lyx orch start` or a task owner names `lyx beam start` or the project owner, under fb5c10c3's rule that messages name the full command, never a shortcut.

- `reconcileAtWake`'s skipped-reconcile notice names `lyx beam start` or `lyx cleanup --apply` as what runs the reconcile after the lift, and `holdRuns`' failed owner pause names "the next wake or `lyx beam start` pauses it again"; `TestWatch`'s two notice assertions follow.
- `hubreconcile`'s way forward names `lyx beam start` among the start verbs that retry the reconcile, and `TestEnsureOrWarn_WarnsWithTheWayForward` asserts it.
- `anchorStopWayForward`'s prime case ends with `lyx beam start` again, and `TestReconcileOrigin_Moves` asserts it.
- The docs: `hibernateengine/doc.go` and `fabricengine/doc.go` name `lyx beam start`; `battencli/cli.go`'s file comment says the beam window is ensured by `lyx beam start`; `PATTERN-pane-text-injection`'s background names `BeamUp`'s start lines; the board skill says owners sit on root and beam projects, a task's owner is its beam's or root's and moves only through `lyx adopt <beam>` or `lyx adopt root`; `docs/overview.md`'s batten line says `lyx beam start` ensures the runs strand.

- `orchengine/launch.go`'s file comment says its two judgments are the ones `orchcli.BeamUp` makes.
- `TestRemovedVerbs_NamedNowhere`, a tier0 scan over the module's production `.go` files, `contracts/stencils` and `plugins/ly/skills`, through `internal/testkit/scankit` as the other cmd/lyx guards scan, fails on any `lyx orch start`, `lyx orch cleanup`, `lyx prime`, `lyx beam promote` or `lyx adopt --prime`, naming the file and line; `docs/research/` is outside the scan, since a bench report records a past run.
  This card is the last that rewords such a site, so the scan passes from here on; its fixture rows show it firing on a planted line and quiet on a clean tree, as the other scans' self-checks do.

**ImpactSummary:** wording only, in messages whose asserting tests are on this card, plus one scan test that pins the acceptance criterion; no behavior changes.

**Commit:** 26: Every way forward names `lyx beam start` where it named `lyx orch start`
