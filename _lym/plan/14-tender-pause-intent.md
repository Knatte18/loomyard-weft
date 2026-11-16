# Card 14 — tender-pause-intent

**Create:**
- `plan:internal/runfsm#RecordPauseIntent` -> `func RecordPauseIntent(location *lymcwd.Location, slug string, intent *shedengine.PauseIntent) (found bool, err error)`
- `plan:internal/runfsm#FetchOwner` -> `func FetchOwner(location *lymcwd.Location, slug string) (owner Owner, offline bool, err error)`
- `plan:internal/runfsm#TestRecordPauseIntent` -> `func TestRecordPauseIntent(t *testing.T)`
- `plan:internal/tendercli#TestResume_RefusesWhatTheActRuleWaitsOrHandsOver` -> `func TestResume_RefusesWhatTheActRuleWaitsOrHandsOver(t *testing.T)`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#PauseReport`
- `internal/tenderengine#ResumeReport`
- `internal/tenderengine#Pause`
- `internal/tenderengine#Resume`
- `internal/tenderengine#TestPause`
- `internal/tenderengine#TestResume`
- `internal/tendercli#HubDeps`
- `internal/tendercli#pauseCmd`
- `internal/tendercli#resumeCmd`
- `internal/tendercli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`

**Extends:**
- `internal/runfsm/doc.go`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`

**Uses:**
- `plan:internal/shedengine#PauseIntent`
- `plan:internal/runfsm#PublishBattenRecord`
- `plan:internal/runfsm#Verdict.Refuse`
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`

**Intent:** `lym tender pause <slug>` writes the run-scope pause intent into the batten record and `lym tender resume <slug>` clears it, each committing and pushing before it returns (D3).
A pause of a run another machine owns is refused, and `lym tender resume <slug>` asks the act rule (D1).

`RecordPauseIntent` writes or, with nil, clears the prime's batten record's `PauseIntent` under the status lock, as card 9's writers do; `found` is false and nothing is written when there is no batten record.
`FetchOwner` pulls the Bolt as battencli's owner check does, then reads `OwnerOf` with `hubgeom.ParentOf`;
an unreachable origin reads the local Bolt and reports `offline`.
`TestRecordPauseIntent` is tier0 in runfsm's `intent_test.go`: write, rewrite and clear over a seeded record, and no write without one.

`tenderengine.Deps` gains `RunOwner func(slug string) (runfsm.Owner, bool, error)` and `PauseIntent func(slug string, intent *shedengine.PauseIntent, push bool) error`.
`Pause` with a slug asks `RunOwner` first and refuses a run another machine owns before it writes anything,
naming that machine and, as way forward, `lym tender pause <slug>` on it or `lym adopt <subject>` here first.
It then writes this machine's pause entry as today, so the run's processes stop, and records the intent with `PauseIntent`, pushing unless `RunOwner` reported offline.
`Resume` with a slug lifts the entry as today and clears the intent the same way.
The reports gain `intent_recorded`, `intent_cleared` and `push_owed`, true when the commit was made offline and its detached push was spawned in its place.
A hub-wide pause or resume writes no intent and asks no owner.
The refusal is reached by a slug whose effective owner on the Bolt is another machine; `TestPause` reaches it.

`HubDeps` wires `RunOwner` to `runfsm.FetchOwner` and `PauseIntent` to `runfsm.RecordPauseIntent` then `runfsm.PublishBattenRecord`, spawning the detached push when the push is skipped offline.
`resumeCmd` with a slug asks `runfsm.Act` over `runfsm.Read` before `tenderengine.Resume`,
setting aside the run's pause intent and shed pause, and passing pause coverage false for the slug's own entries it is about to lift;
a `wait` or `hand_over` refuses through `Verdict.Refuse` with `lym tender resume <slug>` as the rerun.
`pauseCmd` reports the owner refusal as an error envelope.
The three doc files gain the intent, the owner refusal and the act check.

Tests: `TestPause` and `TestResume` (tier0) gain rows for the owner refusal with nothing written, the intent recorded after the entry, the offline push owed, and the intent cleared on resume;
`TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (tier1) asserts the intent is on the batten record's commit after a slug pause and gone after the resume;
`TestResume_RefusesWhatTheActRuleWaitsOrHandsOver` is tier1 in `cli_test.go`: a slug whose record carries a cancel intent is refused with its way forward and keeps its entry.

**ImpactSummary:** Two new seams on tender's engine, used by slug-scoped pause and resume only; hub-wide verbs and the revive path are unchanged.

**Commit:** 14: tender-pause-intent
