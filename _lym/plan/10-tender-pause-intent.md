# Card 10 — tender-pause-intent

**Create:**
- `plan:internal/tendercli#TestPause_PutsTheIntentOnOriginAndRefusesARunOwnedElsewhere` -> `func TestPause_PutsTheIntentOnOriginAndRefusesARunOwnedElsewhere(t *testing.T)`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#Pause`
- `internal/tenderengine#Resume`
- `internal/tenderengine#TestPause`
- `internal/tenderengine#TestResume`
- `internal/tendercli#pauseCmd`
- `internal/tendercli#resumeCmd`

**Extends:**
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`

**Uses:**
- `internal/tendercli#HubDeps`
- `internal/tendercli#PausePair`
- `internal/tendercli#PauseOwnedElsewhere`
- `internal/fabricengine#PullHub`
- `plan:internal/runfsm#ReadOwner`
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#Verdict.Refusal`
- `plan:internal/runfsm#UpdateIntents`
- `plan:internal/runfsm#PublishIntents`

**Intent:**
`lym tender pause <slug>` writes the run-scope pause intent and `lym tender resume <slug>` clears it, each committing and pushing before it returns (D3); resume asks the act rule first (D1).
`tenderengine.Deps` gains three seams, each nil by default so `PausePair`, `PauseOwnedElsewhere` and a hub-scope pause write no intent and refuse nothing new:
`RunOwner`, which says whether another machine owns the slug and names it and the adopt subject;
`PauseIntent`, which writes or clears the slug's intent and publishes it;
`ResumeAct`, which returns the act rule's refusal for the slug, or nil.
`Pause` with a slug refuses before it writes anything when `RunOwner` says another machine owns the run, naming that machine and, as way forward, `lym tender pause <slug>` there or `lym adopt <subject>` here first; it then writes this machine's pause record entry as today, so the run's processes stop, then the intent.
`Resume` with a slug calls `ResumeAct` before it lifts anything and refuses with its text; it then clears the intent and lifts the entries as today.
`pauseCmd` and `resumeCmd` set the seams for a slug: `RunOwner` pulls the Bolt first with `fabricengine.PullHub` and reads `runfsm.ReadOwner`, falling back to the local Bolt when origin is unreachable (D3);
`PauseIntent` is `runfsm.UpdateIntents` setting `PauseIntent{By, Machine, At}` or clearing it, then `runfsm.PublishIntents`; an unpushed intent stays committed here and the verb's report says the push is owed;
`ResumeAct` is `runfsm.Act` over `runfsm.Read` with `Aside{OwnPause: true, ShedPause: true}` and the owner input from `RunOwner`, refusing with `Verdict.Refusal(slug, "lym tender resume <slug>")`.
A pause or resume of a slug with no batten record writes no intent, as `UpdateIntents` writes nothing there.
The new refusals add checks only: neither seam lets a pause or a resume through that was refused before.
Tests: `TestPause` and `TestResume` (tier0) gain rows over fake seams: the owner refusal writes no entry and names its way forward, the intent seam is called after the entry is on disk, and a resume the act rule refuses lifts nothing.
`TestPause_PutsTheIntentOnOriginAndRefusesARunOwnedElsewhere` (tier1) runs the verbs over a `hubforge` hub: the pause intent is on origin's records branch after `lym tender pause <slug>` and gone after `lym tender resume <slug>`, and a slug the Bolt gives another machine is refused naming it.
The docs gain the intent and the owner refusal.

**ImpactSummary:** the slug-scope tender verbs write and push the run's pause intent, refuse a run another machine owns, and resume only what the act rule lets act.

**Commit:** 10: lym tender pause and resume keep the run's pause intent on origin
