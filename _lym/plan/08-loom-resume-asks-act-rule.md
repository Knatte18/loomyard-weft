# Card 8 — loom-resume-asks-act-rule

**Create:**
- `plan:internal/loomcli#TestResumeVerb_RefusesWhatTheActRuleWaitsOrHandsOver` -> `func TestResumeVerb_RefusesWhatTheActRuleWaitsOrHandsOver(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#loomCLI.resumeCmd`

**Uses:**
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#OwnerOf`
- `plan:internal/runfsm#Verdict.Refuse`
- `internal/loomcli#newResumeVerbReceiver`
- `internal/loomcli/doc.go`

**Intent:** `lym loom resume` asks the act rule before it wakes a driver, and refuses `wait` and `hand_over` naming the way forward (D1).

`loomCLI` gains an `actOf` seam, `func(l *lymcwd.Location, slug string) (runfsm.Verdict, error)`; a nil seam reads as `act`, as the nil `ownerOf` reads as owned here.
`newLoomCLI` fills it with `runfsm.Read` and `runfsm.Act` for the task worktree's slug, with the owner from `OwnerOf`,
pause coverage from the hub's pause record, and the shed-pause aside D1's asides give `lym loom resume`.
`resumeCmd` asks it after its covered and owned-elsewhere refusals, which keep their kinds, and before the bootstrap lock;
a refusal is an error envelope whose text is `Verdict.Refuse`'s with `run "lym loom resume" again` as the rerun.
Its `Long` help names the refusal.
The input that reaches the refusal past the earlier checks: a run whose batten record carries a cancel intent.

The new test is tier0 in `resume_test.go`, built on `newResumeVerbReceiver` with a fake `actOf`:
a `wait` verdict and a `hand_over` verdict each refuse with the way forward and leave the park marker in place, and an `act` verdict reaches the existing resume path.

**ImpactSummary:** One new seam on the loom CLI receiver, read by `resume` only.

**Commit:** 8: loom-resume-asks-act-rule
