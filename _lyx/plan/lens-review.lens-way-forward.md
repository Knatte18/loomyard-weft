```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan in `_lyx/plan` (each refusal, stop or recovery step a card adds, the next step it names, and whether that step can succeed).

**Findings**

1. MEDIUM — Card 12, the accept refusal at the budget below the hard cap (D8).
   Once the budget is reached, a recorded `accept` over a refused judgment is retired and the round re-escalates without a re-judge (card 12, "With a refusal at the budget below the hard cap, `retireContinue` retires the accept … and `escalateWith` re-escalates").
   The refused verdict and ledger stay on disk unchanged, so every later `accept` hits the same refusal and re-escalates again.
   The re-escalation's Reason (`escalationReason`, below the hard cap it "offers accept and continue") and the fragment ("an `accept` passes the round … once a judgment is admitted") still offer `accept`, but from that state `accept` can never pass; only `continue` moves the run.
   The re-escalation should say that `accept` re-escalates while the refusal holds and name `continue` as the way forward, or the round should be re-judged budget-exempt.
   Evidence: card 12 lines 70–71 and 86; `internal/shedadapters#Bouncer.escalationReason` (bouncer.go:1501-1504).

2. MEDIUM — Card 12, the hard-cap `Halt` at Discussion-Bouncer (D8).
   The halt leaves the run `blocked` at a Bouncer row that is not a landing halt.
   `Eligibility` admits every such blocked stop, so the arbiter or the Go driver's escalation can resume it, re-judging the round each time.
   D8 bounds that loop with "each resume costs one judge spawn and the operator's resume", but no card keeps it the operator's: card 14 changes only the awaiting `decision` cause.
   Evidence: `internal/arbiterstop/arbiterstop.go:66-70` (a `StateBlocked` stop returns nil unless `LandingHalt`); card 14's Intent.

3. MEDIUM — Card 16, a landing halt left from `Merge-In` (D11 carry-across).
   `HandsBack` is to match `parentAlignName` only.
   Batten and arbiterstop read the task's status file directly, before any loom verb's `armAt` runs `renameRetiredRows`.
   So a run halted on a landing stop at `Merge-In` before the upgrade no longer reads as a hand-back: batten and the arbiter treat it as an ordinary blocked stop the arbiter may settle.
   The way forward that stop's Reason gives the orch (fix on the task branch, then `lyx loom resume`) is then bypassed.
   `HandsBack` should also match `retiredMergeInName`.
   Evidence: `internal/landingshed/handback.go:17`; `internal/arbiterstop/arbiterstop.go:107`; `internal/battencli/wire.go:975`.

4. MEDIUM — Card 16, the attempt-record decode refusal.
   `readAttempts` is to fall back to the old `Merge-In-fixer-attempts.json` when its own record is absent.
   Its decode refusal says "remove &lt;path&gt;", and today `path` is the row's own `FixerAttemptsPath(scratch, mergeInName)` (mergein.go:283-287).
   The card does not say the refusal names the file it actually read.
   If the old record is the undecodable one, the message names the new, absent path; removing it changes nothing and the refusal repeats.
   Evidence: `internal/landingshed/mergein.go:283-287`; card 16, "Attempt record".

5. LOW — Card 12, the `bouncer-fragment-decision` text at or past the hard cap.
   The fragment's way forward always names `lyx loom circling continue {{.slug}}`.
   At or past the hard cap the circling verb refuses `continue` while the escalation is pending, and the capped decide step offers only `accept`.
   So the brief offers a step the verb refuses.
   The verb's own refusal does name `accept` or raising the cap, so the run still moves.
   Evidence: card 12 line 84; `internal/loomcli/circling.go` (`continue` hard-cap refusal at lines 96-108).

**Per refusal, the next step, and whether it progresses**
- Card 12, a pending accept refused below the budget → `rejudge`, then the recorded accept again: progresses.
- Card 12, a pending accept refused at the budget below the hard cap → re-escalate, then `accept` or `continue`: `accept` does not progress (finding 1); `continue` does.
- Card 12, a pending accept refused at the hard cap at the budget → `Halt` naming `lyx loom resume`, which re-judges: progresses, but the resume is not bound to the operator (finding 2).
- Card 12, a pending continue with a dropped key → `rejudge`, and at the budget Shed's budget block, as today: progresses as before.
- Card 12, the `decision` escalation → `lyx loom decision add`, then circling, then `lyx loom resume`: progresses below the hard cap; at the cap the `continue` text is refused (finding 5).
- Card 14, the arbiter's `decision` refusal → `HandBackWayForward` to the parent: progresses.
- Card 10, a darn writer with a parent → `SendMessage` and wait, re-reading the board on a reply: progresses; with no parent → stop report, the gate exhausts, then a board note and `lyx loom resume` respawns the writer: progresses.
- Card 16, the attempt-record decode refusal → remove the named file: does not progress when the old record is the bad one (finding 4); a landing halt from `Merge-In` → `HandsBack`: misread (finding 3).
- Card 4, a run halted at the old reject cap → `lyx loom resume`: progresses.
- Card 6, a retired key → reconcile drops it, and run settings migrate at read time: progresses.

**Tool calls**
- `internal/landingshed/handback.go` and `internal/loomcli/start.go:150-170`: the hand-back predicate's body and the landing-producer check, which my context did not hold.
- `start.go`, `resume.go` and `runstart.go` for the mid-merge check, plus all `HandsBack` callers: to learn whether batten and arbiterstop read the status before the rename.
- `internal/loomcli/decision.go`: the run-state and caller refusals of `lyx loom decision add`, to confirm the escalation's first step can succeed while the run awaits at Discussion-Bouncer.
