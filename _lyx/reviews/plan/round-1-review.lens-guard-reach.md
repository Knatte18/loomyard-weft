```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: checking whether each guard, refusal and test row the plan adds can be reached from a real entry point.

**Per guard, reachable or not**
- Card 3, `normalizeTask` refusing `recipe: pawl` on a task, and `checkProjectFields` refusing any other recipe on a project: reachable through `lyx board upsert`.
- Card 3, `checkOwnerPayload` naming the adopt subject: reachable through an upsert carrying `owner_machine`.
- Card 3, a `Claim` reported `Lost`: reachable only by a race, since batten's start owner refusal reads first; it is a backstop and `SetOwners` tests it directly.
- Card 4, `namedEntries` refusing a task, a grouping project, a note or an unknown slug: reachable through `lyx adopt <slug>`.
- Card 5, `BeamRunLockedError` and the `RunHolds` readers: reachable through `lyx board upsert`, prune or merge at Beam-Await. `rename.go:44` keeps `IsRunStatus`, so a slug rename stays locked, which is consistent.
- Card 7, `EnsureLive` refusing a named beam whose hold stands: reachable from Beam-Await's `StartDobby`, `lyx beam start` and adopt.
- Card 7, `EnsureLive` refusing a named slug that is no local beam pair: no production caller can name such a slug once card 24 has landed. See finding 1.
- Card 7, `EnsureLive` refusing a named beam another machine owns: reachable only by a race, since `BeamUp`'s step-2 owner refusal runs first for start, adopt and Beam-Up.
- Card 7, `ensureParentDobby` refusing a held parent: reachable, but both call sites only log it and put it in a warning field after the pass (`cli.go:459`, `drive.go:107`). It is a warning, not a stop.
- Card 8, the watcher's owner stop, failed read and failed stop: reachable on every tick.
- Card 10, `beamUpRefusal` (owner, anchor move, held beam): reachable from `lyx beam start`, because card 11's `resolveBeamTarget` does not judge the hold.
- Card 11, `resolveBeamTarget`'s refusals (outside a hub, task pair, unknown slug, slug with `--all`): reachable, because `startCmd` and `stopCmd` bypass `runVerb` and so `ResolveBattenPrime`.
- Card 12, a refusal that exits non-zero and attaches nothing: reachable through the existing stop-on-failure dispatch.
- Card 14, `landCheck`: reachable for a member whose Bolt update lands after the re-check; the nil-seam and read-error rows are reachable.
- Card 15, Beam-Await's interrupted-landing branch: reachable. After a crash, a respawned driver at `running · Beam-Await` writes no board status before `Call`. The resume write in step 3b covers only paused, blocked, failed or awaiting states (`internal/shedengine/run.go`, around lines 285–290), and `CommitStatus` fires only on persist (`run.go:904-907`), so the entry still names Parent-Align.
- Card 15, the other Beam-Await branches (another run's request, a failed re-check, a failed restart, a failed status write, another machine's ownership with a yield, a failed `StopDobby` or `Recheck`, a failed `Notify`): all reachable through the seams.
- Card 18, `memberHoldRefusal` after `Add`: reachable when a pass clears `beamForkPrecondition` before the accept writes Parent-Align.
- Card 18, `memberHoldRefusal` over a complete pair: reachable through a hand `lyx batten run <member>` from the prime, which `ResolveBattenPrime` passes. `ensureDobby` runs only after the pass and only warns (`drive.go:107`), so nothing upstream rejects it. From the beam worktree, card 5's `ResolveBattenPrime` hold refuses first.
- Card 18, the grouping-project refusal, the missing parent pair refusal and the draft skip: reachable. Cancel writes `draft` (`cancelcmd.go:461`), so the skip is needed.
- Card 19, `waitLive` at Beam-Await: reachable.
- Card 20, `refuseBeamCancel` (members remain, origin unreachable): reachable through `lyx batten cancel <beam>`.
- Card 21, `refuseCreateHere`: reachable from the prime for a nested beam, or from a beam for a root project, since `ResolveBattenPrime` passes any prime or beam. The lost owner write and lost root claim are reachable by race.
- Card 22, every land refusal: reachable. On the old machine after an adopt elsewhere, the yielding driver leaves the status at `running · Beam-Await`, so the owner refusal is reached after the Beam-Await check.
- Card 23, `landNotice`: reachable on every tick (`loop.go:265` and on).
- Card 25, the slug fallbacks: reachable through nil or failing seams.
- Cards 2, 6, 9, 13, 16 and 26 add no runtime guard beyond fail-loud wiring or scans.

**Findings**

1. LOW, class `design`. Card 7 (`internal/dobbyengine#EnsureLive`), together with cards 10, 11 and 24.
   - Summary: after card 24, no production caller can reach `EnsureLive`'s refusal of a named slug that is no local beam pair; only `TestEnsureLive` reaches it.
   - Why: `BeamUp` names its own resolved worktree (card 10), `--all` names `OwnedWorktrees` (card 11), Beam-Up and Beam-Await name the local beam (card 17), and the anchor move names dobbies it ended (card 7).
   - Card 24 swaps adopt's `StartDobbies` for `BeamUp`, which removes the one caller that could pass a beam with no pair here, an adopted beam whose pair was not added.
   - The way forward it names, `lyx beam create <slug>`, can then never be shown.
   - Evidence: plan cards 7, 10, 11, 17 and 24; `internal/dobbyengine/start.go:87`.

2. NIT, class `consistency`. Card 7 (`internal/battencli#ensureParentDobby`).
   - Summary: the card calls the held-parent case "a refusal naming `lyx batten status <parent>`", but both callers only log it and put it in a warning field after the pass or first-pass spawn.
   - It stops nothing; it only reports.
   - Evidence: `internal/battencli/cli.go:459-462`, `internal/battencli/drive.go:107-109`.

**Tool calls**
- `grep` in `battencli` for the `ensureDobby`, `beamForkPrecondition` and `ResolveBattenPrime` call sites: I needed the pass order, which my context lacked.
- `sed` of `cli.go:440-470`, `drive.go:90-125` and `beamfork.go:20-46`: to see whether the parent-dobby check runs before Worktree-Create and whether it refuses or warns.
- `grep` for board run-status writers: to find what writes the entry's `running · <producer>`, which bears on the interrupted-landing branch.
- `sed` of `loomcli/wiring.go:180-230`: to see when the commit-status seam writes the board.
- `grep` in `shedengine` for `CommitStatus`: to see whether it fires only on persist.
- `sed` of `shedengine/run.go:198-300` (step comments): to see whether a respawned driver persists status before `Call`.
