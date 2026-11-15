# The orch ends its own earlier Claude process and resumes its session

On both recorded adopts, the orch's earlier Claude session stayed alive on the adopting machine after its strand was removed.
It held the transcript, so the resume check refused, and `internal/orchcli/start.go` fell back to a fresh launch on any resume error.
This change makes the orch start recognise its own earlier process, end it and resume the same session id.
It launches fresh only when this machine holds no transcript of the session.
It also makes the orch watcher end every process this machine runs for a subject another machine now owns (D9, D10).

## What changes

- **The orch's Claude process is registered.**
  After each launch, `BeamUp` reads the live provider process holding the started session and registers it with `runproc.RegisterOrch` under the new `runproc.RoleOrch`.
  The owner is the beam's slug, or `runproc.HubOwner` for the prime, the same owner the worktree's dobby uses.
  The record outlives the strand's removal and is pruned once the process dies.
  A process that cannot be found or registered is a warning on the report, and the launch stands.
- **A live holder is ended only when it is ours.**
  `claudeengine.CheckResume` now refuses a held session with a `*shuttleengine.SessionHeldError` naming the pid.
  When `runproc.IsRegistered` names that pid as this worktree's orch, the start ends it through `runproc.EndRole` and resumes the same session id.
  Any other holder refuses the start, naming its pid, with the way forward "exit the session of process N, then run `lym beam start`".
  The start launches nothing fresh over it.
  An orch the pre-upgrade lym launched is unregistered, so the first start after the upgrade refuses over it once.
- **Fresh launch only without a local transcript.**
  A missing transcript now refuses with an error wrapping `shuttleengine.ErrNoTranscript`.
  Only that refusal launches fresh, through shuttle's existing carry-over.
  The report's warnings say the launch is fresh because this machine holds no transcript, with the count of carried instructions, and `abandoned_session` names the session given up.
  Every other failed resume fails the start with a way forward, where it used to launch fresh.
- **The watcher ends a moved subject's processes.**
  `orchengine.OwnerWatch` gains `End`, which `ownerTick` calls before it stops the strand; a failed end is the tick's error and the next tick retries.
  `orchcli` wires it to `endSubjectProcesses`, which picks the registered owners whose `boardengine.AdoptSubject` is the beam, or the root for the prime.
  It hands them to the new `runproc.EndSubject`.
  `EndSubject` ends each run's processes, then the subject's own records with the orch's Claude process, and the dobby last.
  It leaves the hub's adopt worker and board syncs alone, since they serve every subject.
  The dobby's handover and the catch-up in `bolt-holds-catch-up` can call the same function.
- **D10 holds as it was.** The session id stays in `orchengine.State` under `.lym/orch/`, which is never tracked, and nothing carries a transcript between machines.

## Supporting changes

- `shuttleengine` gains the optional `SessionHolderFinder` capability, which Claude implements over its session registry, plus `Run.SessionHolder` and `Run.CarriedInstructions`.
  The registry rule now lives in one `liveHolder` helper shared by the resume check and the holder read, and `sessionRegistryDir` replaces the path the prober and the check each built.
- `lym fabric anchor` adds `runproc.RoleOrch` to the roles its dobby end spares.
  Without it, the new orch record under the dobby's owner would end the orch on every anchor move, which it never did before.
- `pattern/PATTERN-child-processes.md` names `EndSubject` beside `End`, `EndSparing` and `EndRole`, and names the orch's registration.
- Package docs of `runproc`, `orchcli`, `orchengine`, `shuttleengine`, `claudeengine` and `fabriccli` follow the behavior.

## Design decisions

- The way forward names `lym beam start`, the verb that brings the orch up; the record's `lym orch start` is no verb.
- The orch record shares the dobby's owner, so `runproc.End` of a beam's slug, as a pair teardown or `tendercli.EndOwnedElsewhere` runs, now ends that beam's orch too.
  That matches D9's "end every process of a subject owned elsewhere".
- `EndSubject` ends the registered processes of a subject's runs.
  A run's agent panes that registered no process stay until the run's own teardown.

## Tests

- `runproc`: `TestRegisterOrch_IsFoundByItsOwnerRoleAndPID` and `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`, which pins the order, the spared roles and that a surviving run process still lets the rest end.
- `claudeengine`: the resume check's rows assert the typed refusals, and `TestSessionHolder` pins the holder read.
- `orchengine`: the owner-check rows assert that End runs before Stop and that a failed end stops nothing.
- `orchcli`: `TestStart_ResumeOverALiveHolder` covers each acceptance line: an own holder ended and the same session resumed, and an unregistered holder refused by pid with nothing launched fresh.
  It also covers a failed end and a resume failing for another cause.
  The no-transcript rows of `TestStart_ResumesTheRecordedSession` assert the fresh launch, its carry-over count and the warning that says so.
  `TestStart_RegistersTheLaunchedOrchProcess` pins the registration and its warnings.
