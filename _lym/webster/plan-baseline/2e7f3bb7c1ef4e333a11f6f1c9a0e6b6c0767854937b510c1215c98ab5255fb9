# Card 12 — loom-resume-act

**Create:**
- `plan:internal/loomcli#loomCLI.refuseByAct` -> `func (c *loomCLI) refuseByAct(ctx context.Context, out io.Writer, retry string) bool`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.resumeCmd`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#TestResumeVerb`
- `internal/loomcli#TestStartAndResume_RefuseARunOwnedElsewhere`

**Uses:**
- `plan:internal/pairteardown#ReadRun`
- `plan:internal/runfsm#Judge`
- `internal/loomcli/doc.go`

**Intent:** `lym loom start` and `lym loom resume` ask the act rule before they wake or spawn a driver (D1).
`loomCLI` gains `runVerdict func(l *lymcwd.Location, slug string) (runfsm.Verdict, error)`, filled where `ownerOf` is filled, over the hub's prime (`primeOfHub`) through `pairteardown.ReadRun` and `runfsm.Judge` with `ShedPause` set aside, since these verbs resume the child's own shed pause.
A nil `runVerdict` checks nothing.

`refuseByAct` asks it, as `refuseOwnedElsewhere` does, only for a run whose status file exists, since a spawn with no status is the fresh bootstrap that a start batten already judged.
It writes the refusal and returns true on any answer but act: a hand-over writes the envelope kind `owned_elsewhere` with the owner, as the owner refusal does; any other wait writes `Verdict.Refusal(slug, retry)`; a read error names its cause and the retry.
`resumeCmd` and `startCmd` call it after their pause and owner refusals, with their own retry, before the bootstrap lock.
Their Long texts add that a run the act rule waits on is refused with its way forward.

`lym loom run` and `lym loom step` do not ask, as they ask neither the pause nor the owner refusal today.
They are role-audience primitives: the driver `lym loom start` or `lym loom resume` spawns, or an external supervisor, drives them after start or resume asked the rule.
Bound: a cancel or pause that lands mid-run stops the run through the cancel's end of its processes and the pause record its processes read, not through these verbs,
and a hand `lym loom step` or `lym loom run` on a run the rule waits drives it one producer or one run further.

Tests, tier0:
- `TestResumeVerb` gains rows over a fake `runVerdict`: a cancelling wait naming the cancel, a terminal cancelled wait naming `--remove` (a kept pair's task worktree, where the verb can run), a read error, and act going on to the state read.
- `TestStartAndResume_RefuseARunOwnedElsewhere` gains a row where the owner reads here and the verdict waits on a cancelling run, refusing both verbs with the cancel's way forward.

**ImpactSummary:** One new refusal step in the start and resume verbs behind a seam; the other loom verbs are untouched, run and step for the reason the Intent gives.
