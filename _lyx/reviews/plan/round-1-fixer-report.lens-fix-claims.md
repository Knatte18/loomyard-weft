```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: checking that each claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM: card 11, step 7's history test and step 10's "whose entry re-runs step 7"**
- **Summary:** step 7 re-runs the verify gate only when the last Stuck's reason, past any `DecisionHaltPrefix`, opens with a `driftengine` halt-opening constant. Nothing says step 10's own Stucks open with those constants, and those are the ones raised when the dirty or verify gate entry spends its attempts. So the claim that a resume from them re-runs step 7's verify half does not follow from the card.
- **Evidence:** the seat producer words a gate-exhaustion Stuck itself, as `Reason: gateFailedReason(chair.Gate, p.gateFindingsPath)` (`internal/shedadapters/multillm.go:161`). That is shedadapters' text, not a `driftengine` opening. Card 11 step 10 says only that this Stuck "keeps the gate's findings file and names its way forward". It never says the reason is reworded to open with step 7's constants. As the card reads now, a resume after the agent's verify entry ran out of attempts fails the history test and compares without re-running the verify gate. That is the gap the round-1 way-forward fix closed.

**LOW: card 11, the F2 row in `TestDriftVerify` ("gateLoopSeats reporting a live chair on entry, the producer resumes it")**
- **Summary:** the new step 2 probes once, then step 9's `MultiLLMProducer.Call` probes again before it resumes. The row has to script a live chair for both probes, and the card does not say so.
- **Evidence:** `MultiLLMProducer.Call` calls `p.seats.Probe(table)` and resumes only on `live.Chair != nil` (`internal/shedadapters/multillm.go:102-112`). `shedfake.SeatRunner` serves its scripted `LiveTables` one per call, and an unscripted probe answers no live seat (`internal/testkit/shedfake/shedfake_test.go:157-171`).

Claims I checked that hold:
- The foreground `RunCapture` command dies with the step: Windows `StartTied` puts it in a parent-held kill-on-close job (`internal/proc/tied_windows.go:14`).
- A live chair skips `prepareFreshSpawn`, so the fresh-spawn hook and `ResetRuns` do not run (`multillm.go:106-120`).
- The step-2 probe matches regardless of the compare log it is told, since `probeSeat` builds its spec with an empty prompt (`seatengine/resume.go:67`, `prompt.go:163-185`).
- `CommitAnchoredPaths` takes `relPaths` (`internal/fabricengine/commitweftpaths.go:100`).
- A cobra command carries one audience (`internal/clihelp/annotations.go`).
- Card 8's verify stencil names `lyx drift capture --side`.

Tool calls:
1. `proc` `StartTied` on Windows and `multillm.go:100-140`: my context lacked the Windows tie and the producer's live-chair branch.
2. `StartTied` file locations and the `shedfake` `Probe` scripting: my context lacked how the fake scripts probes.
3. `seatengine` `probeSeat` and `seatSpec`: my context lacked what a probe matches on.
4. `shuttleengine` `ProbeGated`: my context lacked whether a gateless probe table changes the match.
5. The gate-exhaustion Stuck wording in `multillm.go`: my context lacked the reason text the seat producer writes.
