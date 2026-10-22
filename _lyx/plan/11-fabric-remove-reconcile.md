# Card 11 — lyx fabric remove reconciles the run first and removes an ended or foreign run locally

**Create:**
- `plan:internal/fabriccli#removeVerdict` -> `func removeVerdict(report fabricengine.OriginReport, slug string, local localReading) (verdict fabricengine.EntryState, refusal error)`
- `plan:internal/fabriccli#localReading` -> `type localReading struct`
- `plan:internal/fabriccli#remover` -> `type remover struct`
- `plan:internal/fabriccli#remover.run` -> `func (r remover) run(ctx context.Context, slug string, force, remote bool) (pairteardown.Result, error)`
- `plan:internal/fabriccli#TestRemoveVerdict` -> `func TestRemoveVerdict(t *testing.T)`
- `plan:internal/fabriccli#TestRemover` -> `func TestRemover(t *testing.T)`
- `plan:internal/pairteardown#Teardown.TaskWorktreeGone` -> `func (t *Teardown) TaskWorktreeGone(slug string) (bool, error)`

**Edit:**
- `internal/fabriccli#runRemoveWithFlag`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`

**Uses:**
- `plan:internal/pauserecord#SetGateExemptAll`
- `plan:internal/runproc#Ended`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#Teardown.OriginSeams`
- `internal/pairteardown#Teardown.taskWorktreeGone`
- `internal/boardengine#ReadOwners`

**Intent:**
D5's verb half.

`remover` carries the seams `runRemoveWithFlag` wires and `TestRemover` fakes: `exemptAll` (`pauserecord.SetGateExemptAll`), `gone` (`Teardown.TaskWorktreeGone`, an exported wrapper of `taskWorktreeGone`), `reconcile` (the run-scope `fabricengine.ReconcileOrigin(l, ReconcileScope{Run: slug}, td.OriginSeams(pairteardown.HubSeamOptions(l)), ReconcileOptions{})`), `local` (the `localReading` below) and `teardown` (`Teardown.Run`).
`remover.run` first calls `exemptAll` and restores it on return, so the removal's own network calls pass under any pause (the bound: this process only; its detached children have their own gates, and `--force` still answers dirtiness only).
A pair whose task worktree is gone takes no reconcile, since a run-scope reconcile refuses a slug with no pair worktree (`originTargets`), and goes on with the empty verdict to D4's gone-worktree path: the refusal probe, `End` for the slug, the session end by exact name.
Otherwise it runs the run-scope reconcile of the slug, as `PATTERN-origin-reconcile` requires; a reconcile error refuses naming its cause and the re-run, with nothing ended.
`removeVerdict` turns the report into the verdict on the slug's entry:
`stopped` refuses with the entry's way forward, then `lyx fabric remove <slug>` again, as every run-scope caller does, with nothing ended;
`ended` and `owned_elsewhere` are returned as the verdict;
`active`, or no entry (a pair with no batten run record), returns the empty verdict, today's path.
With `report.Unreachable` the local Bolt decides, as for every other reconcile caller: `localReading` carries whether the local board's effective owner of the slug is another machine (read with `boardengine.ReadOwners` and `OwnedElsewhere`) and whether the local entry reads done, which give `owned_elsewhere` and `ended`.
The run scope never tears down its own slug (`reconcileEnded` skips `scope.Run`), so the report never shows the pair already removed.

An `ended` or `owned_elsewhere` verdict removes the pair through `pairteardown`'s local mode, `Request{Slug, Local: true, Verdict: v, RefuseWhenBusy: true, Force: force}` with a zero quiet wait: no archive-tag push, no remote branch step, no board write, and a machine that does not own the run pushes no archive tag of stale records (#741).
A `--remote` it drops is named in the result's `RemoteSkippedReason`: "the origin reconcile read the entry <verdict>, so the removal was local and --remote was dropped".
Any other verdict runs today's request, `RemoveQuietWait` with `RefuseWhenBusy` and the typed `--force` and `--remote`.

`removeFields` gains `ended_processes`, the list of `{pid, role}` the session end reported, empty when none, so the paused-run removal names the driver, loop and batten it ended.
`TestRemoveVerdict` rows: stopped refuses naming the way forward and the re-run; ended and owned_elsewhere pass through; active and an absent entry give the empty verdict; the parent pair's entry of a run-scope report is not read as the slug's; unreachable with a local foreign owner gives `owned_elsewhere`, with a local done entry gives `ended`, with neither gives the empty verdict.
`TestRemover` rows over fake seams, each asserting the call order and the request the fake teardown saw: an ended verdict, with `--remote`, runs the local request and names the dropped `--remote`; an owned_elsewhere verdict runs the local request; an active verdict runs today's request with `RemoveQuietWait`; a reconcile error refuses with no teardown call; a stopped verdict refuses with no teardown call; a gone task worktree runs no reconcile and today's request; the exemption is taken before the reconcile and restored after the teardown on every row, the refusing ones included.
`TestRemoveFields` gains a row with two ended processes and asserts the default row's `ended_processes` is an empty list.
`Command`'s `remove` help text and `doc.go` state the reconcile, the gone-worktree path, the local removal of an ended or foreign run, the stopped refusal, the gate exemption and `ended_processes`.

**ImpactSummary:** fabric remove's handler is split into a seam-driven remover with a reconcile and a verdict branch; removeFields gains one key; pairteardown gains one exported read; no other caller of pairteardown changes.

**Commit:** 11: lyx fabric remove reconciles first and removes an ended or foreign run locally
