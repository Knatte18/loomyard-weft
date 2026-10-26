```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review: each refusal and recovery step the cards add, checked for whether its named next step can succeed from the state it leaves.

**BLOCKING — card 18 (D9 member hold read), `memberHoldRefusal`: neither named next step can succeed.**
- The refusal says the member starts "once the beam has landed or after `lyx board upsert` moves its `project` off the beam".
- The beam cannot land while this member is open. The refused member keeps its claim, `running · Worktree-Create` (`internal/battencli/owner.go` `claimBoardEntry`), so it is still an open member. `BeamLandRefusal` counts open members (card 13's `beamMembers`), so Beam-Await's re-check and Finalize's land check both refuse the beam.
- The upsert is refused by the run lock. The claim status is a run status, `RunHolds` is true for a task (card 5), and `checkRunLocks` refuses a `project` change (`internal/boardengine/runlock.go` `scopeChanged` compares `Project`). D9 itself says "which the run lock would refuse for a run-held member".
- The refused run is halted and dobby gives a halted run no pass (`internal/dobbyengine/doc.go` "A halted or paused batten run gets no pass"). Once Beam-Await's failed re-check lifts the hold, nothing names the hand `lyx batten run <member>` that resumes it.
- The way forward should name: `lyx batten run <member>` once `lyx batten status <beam>` shows the run back at Beam-Await, or `lyx batten cancel <member>` and then the board move.

**BLOCKING — card 4 (D6), `takeOverSlug`'s stop: the way forward names a task slug that card 4 makes adopt refuse.**
- A moved run's slug "stops as a named slug stops today".
- Today's text is "`lyx adopt %s --keep-local` records the owner over it, and `lyx adopt %s --drop-local` …", with the stopped slug (`internal/adoptcli/adopt.go` `takeOverSlug`).
- For a moved task, card 4's own `namedEntries` now refuses `lyx adopt <task>`.
- The intent never rewords this text to `lyx adopt <adopted beam or root> --keep-local|--drop-local`, and no `TestAdopt` row asserts it.

**MEDIUM — card 15 (D9 Beam-Await accept): a `Recheck` read error and a `StopDobby` failure have no stated branch.**
- Both happen after Parent-Align is written and, for the re-check, after the marker is written.
- The state left is: landing hold standing, dobby possibly stopped, marker standing.
- What the row does is unspecified (Stuck, error, or treat as a failed re-check), so whether the request is kept and the hold lifted is undefined.
- `lyx beam start` and member forks are refused while the hold stands, so the recovery is unknown. No `TestBeamAwait` row reaches either branch.

**MEDIUM — card 20 (D11 beam cancel): the named re-start `lyx beam create <slug>` meets an unnamed draft refusal.**
- The cancel's `ClearClaim` writes `draft` on the beam project (`internal/battencli/cancelcmd.go` `clearRunClaim`).
- Card 21's create then starts a first pass, and card 18's guard lets a fresh beam project fall through to "a task's … checks".
- `startGuard.refuse`'s `Draft` probe reads any entry's draft status (`internal/battencli/startguard.go` `hubStartGuard` `Draft`), so the first pass is refused as "a draft task", needing a release that neither card names.
- Card 18 should exempt a beam project from the draft check, or card 20 should not leave a beam project at draft. No test row covers cancel then create.

**MEDIUM — card 21 (D10), `refuseCreateHere`: it names a path that does not exist on this machine.**
- It refuses anywhere but the parent worktree, "naming that worktree's path".
- For a nested beam whose parent beam's pair is absent here, that path does not exist.
- Card 21 deletes `requireParentBeamPair`, which named `lyx beam create <parent>` or `lyx adopt <parent>`. Card 18's Worktree-Create copy of that refusal is never reached, because create refuses first.
- The way forward should name the parent's create or adopt when its pair is absent. No `TestCreate` row covers it.

**LOW — card 14 (D9 Finalize land halt): `landHaltReason` cannot name the machine it promises to name.**
- It promises to name "the machine holding the pair when the run is in flight and the pair is not here".
- `LandBlocker` carries only the slug, `Pair`, `HasPair` and `Running`, so no machine can be named.

**LOW — cards 15 and 17 (D9 Beam-Await failure post): a failed `Notify` is unspecified.**
- The failed re-check removes the request, then posts through `orchcli.NotifyOrchAt` to the beam's own orch.
- If no orch strand is recorded, or the post fails, the failure and its blockers reach no one.
- `lyx beam land` had already returned ok, so the operator never learns the request was dropped.

Tool calls:
- One read of `internal/battencli/cancelcmd.go` `clearRunClaim`, to learn which status a cancel leaves on the entry; my context did not show its body.
