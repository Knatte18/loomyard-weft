# Card 14 — tender-run-pause-intent

**Create:**
- `plan:internal/tendercli#TestPauseAndResume_RecordTheRunPauseIntentOnOrigin` -> `func TestPauseAndResume_RecordTheRunPauseIntentOnOrigin(t *testing.T)`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#PauseRequest`
- `internal/tenderengine#ResumeRequest`
- `internal/tenderengine#PauseReport`
- `internal/tenderengine#ResumeReport`
- `internal/tenderengine#Pause`
- `internal/tenderengine#Resume`
- `internal/tenderengine#TestPause`
- `internal/tenderengine#TestResume`
- `internal/tendercli#HubDeps`
- `internal/tendercli#pauseCmd`
- `internal/tendercli#resumeCmd`

**Uses:**
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`
- `plan:internal/shedrun#UpdateRunRecord`
- `plan:internal/shedengine#PauseIntent`
- `plan:internal/runfsm#Asides`
- `plan:internal/pairteardown#PublishBattenRecord`
- `plan:internal/pairteardown#AskAct`
- `plan:internal/pairteardown#ActRefusal`

**ImpactSummary:** `lym tender pause <slug>` and `lym tender resume <slug>` write and clear the run's tracked pause intent and push it, refuse a run another machine owns, and resume asks the act rule; the hub-wide verbs and every other caller of `Pause`, `Resume` and `Lift` are unchanged.

**Intent:**
Give a pair's pause a run-scope pause intent in the batten record, written only by the run's owner (D3).

`PauseRequest` and `ResumeRequest` gain `Run`, set only by the verbs for a slug; `PausePair`, `PauseOwnedElsewhere` and the cancel's `Lift` leave it unset and change nothing.
`Deps` gains three seams, each nil-safe as its others are:
- `RunOwner func(slug string) (owner, subject string, err error)`: the machine that owns the run when it is another one, and the adopt subject.
- `SetRunPause func(slug string, paused bool) (pushed bool, err error)`.
- `RunAct func(slug string) error`.
For a `Run` request, `Pause` and `Resume` first ask `RunOwner` and refuse a run another machine owns, naming that machine.
The way forward is the same verb on that machine, or `lym adopt <subject>` on this one first; nothing is written.
`Pause` then records this machine's pause entry as today, so the run's processes stop here, and calls `SetRunPause(slug, true)`.
`Resume` calls `RunAct` and refuses with its error, then calls `SetRunPause(slug, false)`, then lifts the entry as today.
`PauseReport` and `ResumeReport` gain `intent_pushed`.

`HubDeps` fills the seams over the prime:
- `RunOwner` pulls the hub's Bolt as `lym batten cancel` does, reads the local Bolt when origin is unreachable, and resolves the effective owner through `hubgeom.ParentOf`.
- `SetRunPause` writes or clears `Status.Pause` through `shedrun.UpdateRunRecord`, with the caller label and the time, and publishes it with `pairteardown.PublishBattenRecord` with push.
  A slug with no batten record writes nothing.
  An unreachable origin reports not pushed and leaves the commit for the next push, so the pause still holds this machine's processes.
- `RunAct` asks `pairteardown.AskAct` with the pause and the shed pause set aside and refuses any answer but act through `pairteardown.ActRefusal`, re-run `lym tender resume <slug>`.
`pauseCmd` and `resumeCmd` set `Run` for a slug, and their help says the run's pause is recorded on origin and that only the owner pauses or resumes a run.

`TestPause` and `TestResume` (tier 0, fakes) gain the rows: an owned-elsewhere refusal that writes no entry and no intent, the intent written after the entry, a resume refused by `RunAct` lifting nothing, and a request without `Run` calling none of the seams.
`TestPauseAndResume_RecordTheRunPauseIntentOnOrigin` (tier 1, hubforge with an origin) asserts `lym tender pause <slug>` puts the pause intent on origin and a pause entry here, and `lym tender resume <slug>` clears both.

**Commit:** 14: lym tender pause and resume of a pair record the run's pause intent on origin
