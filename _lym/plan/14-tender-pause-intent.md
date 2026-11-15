# Card 14 — tender-pause-intent

**Create:**
- `plan:internal/tendercli#runIntents` -> `type runIntents struct`
- `plan:internal/tendercli#hubRunIntents` -> `func hubRunIntents(location *lymcwd.Location, slug string) (runIntents, error)`

**Edit:**
- `internal/tenderengine#PauseRequest`
- `internal/tenderengine#Pause`
- `internal/tenderengine#ResumeRequest`
- `internal/tenderengine#Resume`
- `internal/tenderengine#TestPause`
- `internal/tenderengine#TestResume`
- `internal/tenderengine/doc.go`
- `internal/tendercli#pauseCmd`
- `internal/tendercli#resumeCmd`
- `internal/tendercli#EndOwnedElsewhere`
- `internal/tendercli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `internal/tendercli/doc.go`

**Uses:**
- `plan:internal/boardengine#PullBolt`
- `plan:internal/shedrun#WritePauseIntent`
- `plan:internal/shedrun#ClearPauseIntent`
- `plan:internal/pairteardown#ReadRun`
- `plan:internal/runfsm#Judge`
- `plan:internal/fabricengine#OriginParentHoldsTaskWork`

**Intent:** `lym tender pause <slug>` and `lym tender resume <slug>` write and clear the run-scope pause intent in the batten record, and refuse a run another machine owns (D3).
Only the two verbs set the new request fields; `PausePair` and `PauseOwnedElsewhere`, lym's own pauses, and every hub-scope pause leave them nil, which changes nothing.

`tenderengine`:
- `PauseRequest` gains `Owner func() (owner, subject string, err error)` and `Record func() error`.
  `Pause` asks `Owner` first; a non-empty owner refuses with no entry written:
  `pause: <slug> is owned by machine <owner>; way forward: run "lym tender pause <slug>" on <owner>, or "lym adopt <subject>" on this machine first`.
  Once the entry is on disk, an already-paused scope included, and before the grace, it calls `Record`; its error ends the pause wrapped with the way forward `fix the cause, then run "lym tender pause <slug>" again`, which records the intent over the entry that stays live, so the run's processes stay held here.
- `ResumeRequest` gains `Verdict func() (runfsm.Verdict, error)` and `Clear func() error`.
  `Resume` asks `Verdict` first, refusing any answer but act with `Verdict.Refusal(slug, "run \"lym tender resume <slug>\" again")` and lifting nothing; then `Clear`, whose error refuses with nothing lifted and the way forward `fix the cause, then run "lym tender resume <slug>" again`; then the lift as today.

`tendercli`:
- `runIntents` holds the four seams, and `hubRunIntents` fills them over the hub's prime:
  `Owner` pulls the Bolt through `boardengine.PullBolt`, reads the local Bolt when origin is unreachable, and returns the effective owner and adopt subject when another machine owns slug, through `hubgeom.ParentOf` and `machinename.Name`;
  a pull that fails otherwise returns its error with the way forward `bring the board level as it names, then run the verb again`;
  `Record` writes the intent, `By` the caller label of the strand and `At` now, through `shedrun.WritePauseIntent` on the prime, commits the prime's `shedrun.StatusRel` for slug, and spawns the prime's status push with slug as its gate slug;
  `Verdict` is `pairteardown.ReadRun` then `runfsm.Judge` with `Pause`, `Cancel`, `ShedPause` and `PushOwed` set aside: the resume lifts its pause entry and drives nothing, and D3 has it act offline (Shared Decisions);
  `Clear` clears the intent through `shedrun.ClearPauseIntent` and commits and pushes as `Record` does.
  A slug with no batten record writes and commits nothing.
- `pauseCmd` asks `Owner` before its no-pair check when a slug is named, so a run another machine owns, which has no pair here, is refused naming that machine rather than the pair list.
  `pauseCmd` and `resumeCmd` set the fields from `hubRunIntents` when a slug is named, and their Long texts say the intent is committed and pushed, and that a run another machine owns is refused naming that machine.
- `EndOwnedElsewhere` reads its parent through `fabricengine.OriginParentHoldsTaskWork`, or `ParentHoldsTaskWork` under the push skip, so origin's fetched parent branch decides whether its Finalize is past the parent-side merge (D1).

Bounds: offline, the pause entry here stops the run's processes and the intent is committed with its push owed; until it lands another machine reads the run unpaused, as D3 states.
A resume offline commits the cleared intent and lifts its entry; the run's own verbs wait on the owed push once the later part fills that input.

Tests:
- `TestPause` (tier0) gains rows: an owner elsewhere refused with no entry; `Record` called once after the entry is on disk, and again for an already-paused scope; a `Record` error returned with its way forward and the entry live; nil fields as today.
- `TestResume` (tier0) gains rows: a wait verdict refusing with nothing lifted; `Clear` called before the lift; a `Clear` error refusing with nothing lifted and its way forward.
- `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (tier1) gains rows over its hub: a slug with a batten record whose pause commits the intent on the prime's records side and whose resume commits it cleared;
  a slug with no batten record whose pause commits nothing;
  a slug the Bolt records as owned by another machine, with no pair here, refused naming that machine;
  and a batten-paused run whose `lym tender resume <slug>` lifts its entry.

**ImpactSummary:** Two request types gain optional seams the two tender verbs fill, the pause verb asks the owner before its pair check, and the owned-elsewhere end reads origin's parent; lym's own pause callers and hub-wide pauses are unchanged.
