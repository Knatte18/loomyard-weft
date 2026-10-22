# Card 11 — lyx fabric remove reconciles the run first and removes an ended or foreign run locally

**Create:**
- `internal/fabriccli#removeVerdict`
- `internal/fabriccli#localReading`
- `internal/fabriccli#remover`
- `internal/fabriccli#remover.run`
- `internal/fabriccli#TestRemoveVerdict`
- `internal/fabriccli#TestRemover`
- `internal/pairteardown#Teardown.TaskWorktreeGone`

**Edit:**
- `internal/fabriccli#runRemoveWithFlag`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/fabriccli#Command`
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/pauserecord#SetGateExemptAll`
- `internal/runproc#Ended`
- `internal/pairteardown#HubSeamOptions`
- `internal/pairteardown#Teardown.OriginSeams`
- `internal/pairteardown#Teardown.taskWorktreeGone`
- `internal/boardengine#ReadOwners`

**Intent:**
D5's verb half.

`remover` carries the seams `runRemoveWithFlag` wires and `TestRemover` fakes: `exemptAll` (`pauserecord.SetGateExemptAll`), `gone` (`Teardown.TaskWorktreeGone`, an exported wrapper of `taskWorktreeGone`), `reconcile` (the run-scope `fabricengine.ReconcileOrigin(l, ReconcileScope{Run: slug}, td.OriginSeams(pairteardown.HubSeamOptions(l)), ReconcileOptions{})`), `local` (the `localReading` below) and `teardown` (`Teardown.Run`).
`remover.run` first calls `exemptAll` and restores it on return, so the removal's own network calls pass under any pause (the bound: this process only; its detached children have their own gates, and `--force` still answers dirtiness only).
A pair whose task worktree is gone takes no reconcile, since a run-scope reconcile refuses a slug with no pair worktree (`originTargets`); the local Bolt decides its verdict through `localReading`, as for an unreachable origin below (`removeVerdict` over an empty report with `Unreachable` set), and it goes on to D4's gone-worktree path: the refusal probe, `End` for the slug, the session end by exact name.
So an `owned_elsewhere` or `ended` reading still removes through the local request, its dropped `--remote` named, and only a pair with neither reading runs today's request.
Otherwise it runs the run-scope reconcile of the slug, as `PATTERN-origin-reconcile` requires; a reconcile error refuses naming its cause and the re-run, with nothing ended.
`removeVerdict` turns the report into the verdict on the slug's entry:
`stopped` refuses with a way forward, then `lyx fabric remove <slug>` again, as every run-scope caller does, with nothing ended;
the way forward comes from `report.Branches`, never from `EntryOutcome.WayForward`: it is the `WayForward` of the first stopping branch that is neither a `BranchGone` branch of the slug nor one of the `BranchBusy` branches the gone-branch exception below leaves out (the slug's own records branch, and a shared branch, reading `BranchBusy`), whether that branch is the slug's own (diverged, dirty, rewound, rewritten) or a shared Bolt or prime-records stop,
since `classifyEntries` fills the entry's way forward with the first stopping branch of the slug in target order, the code branch before its records sibling and the slug's own stop before the shared one, so a gone code branch listed first would give the entry the gone text, which names the refused verb;
the one exception is a stop whose every stopping branch of the slug (a `report.Branches` entry with `Slug` equal to the slug and a stopping `State`, leaving out the slug's own records branch reading `BranchBusy`, which `classifyEntries` skips in the run scope) reads `fabricengine.BranchGone`, with no shared branch, the Bolt or a prime's records branch, stopping other than by `BranchBusy`, which `classifyEntries` does not count as a shared stop: it returns the empty verdict, today's request, since a gone branch's way forward (`FailedPushWayForward` for the gone class, reached through `stopWayForward`) is "`lyx fabric remove` of the pair ... removes the pair and clears this record", so refusing it would name the refused verb and loop;
a gone branch beside any other stopping branch still refuses, naming that other branch's way forward;
`ended` and `owned_elsewhere` are returned as the verdict;
`active`, or no entry (a pair with no batten run record), returns the empty verdict, today's path.
With `report.Unreachable` the local Bolt decides, as for every other reconcile caller: `localReading` carries whether the local board's effective owner of the slug is another machine (read with `boardengine.ReadOwners` and `OwnedElsewhere`) and whether the local entry reads done, which give `owned_elsewhere` and `ended`.
The run scope never tears down its own slug (`reconcileEnded` skips `scope.Run`), so the report never shows the pair already removed.

An `ended` or `owned_elsewhere` verdict removes the pair through `pairteardown`'s local mode, `Request{Slug, Local: true, Verdict: v, RefuseWhenBusy: true, Force: force}` with a zero quiet wait: no archive-tag push, no remote branch step, no board write, and a machine that does not own the run pushes no archive tag of stale records (#741).
A `--remote` it drops is named in the result's `RemoteSkippedReason`: "the origin reconcile read the entry <verdict>, so the removal was local and --remote was dropped".
Any other verdict runs today's request, `RemoveQuietWait` with `RefuseWhenBusy` and the typed `--force` and `--remote`.

`removeFields` gains `ended_processes`, the list of `{pid, role}` the session end reported, empty when none, so the paused-run removal names the driver, loop and batten it ended.
`TestRemoveVerdict` rows: stopped refuses naming the way forward and the re-run; stopped only by a gone code branch of the slug gives the empty verdict, and so does the same stop beside a busy Bolt branch; stopped by a gone branch and a diverged one still refuses naming the diverged branch's way forward, and so do the rows that list the gone branch first: a gone code branch then a diverged records branch of the slug, a gone code branch then a diverged Bolt branch, and a gone code branch, then the slug's own records branch reading busy, then a diverged Bolt branch, each refusal containing the other branch's `WayForward` text and not the gone branch's text (its clause "removes the pair and clears this record"), though the entry's `WayForward` holds the gone text; the refusal's closing "then `lyx fabric remove <slug>` again" is expected in every row; ended and owned_elsewhere pass through; active and an absent entry give the empty verdict; the parent pair's entry of a run-scope report is not read as the slug's; unreachable with a local foreign owner gives `owned_elsewhere`, with a local done entry gives `ended`, with neither gives the empty verdict.
`TestRemover` rows over fake seams, each asserting the call order and the request the fake teardown saw: an ended verdict, with `--remote`, runs the local request and names the dropped `--remote`; an owned_elsewhere verdict runs the local request; an active verdict runs today's request with `RemoveQuietWait`; a reconcile error refuses with no teardown call; a stopped verdict refuses with no teardown call; a gone-branch stop runs today's request with `RemoveQuietWait` and no refusal; a gone task worktree with a local foreign owner runs no reconcile and the local request; a gone task worktree with neither local reading runs no reconcile and today's request; the exemption is taken before the reconcile and restored after the teardown on every row, the refusing ones included.
`TestRemoveFields` gains a row with two ended processes and asserts the default row's `ended_processes` is an empty list.
`Command`'s `remove` help text and `doc.go` state the reconcile, the gone-worktree path and its local reading, the local removal of an ended or foreign run, the stopped refusal and its gone-branch exception, the gate exemption and `ended_processes`.

**ImpactSummary:** fabric remove's handler is split into a seam-driven remover with a reconcile and a verdict branch; removeFields gains one key; pairteardown gains one exported read; no other caller of pairteardown changes.

**Commit:** 11: lyx fabric remove reconciles first and removes an ended or foreign run locally
