# Card 12 — batten-run-asks-the-act-rule

**Create:**
- `plan:internal/battencli#battenCLI.refuseByAct` -> `func (c *battenCLI) refuseByAct() error`
- `plan:internal/battencli#TestRefuseByAct` -> `func TestRefuseByAct(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.battenPreRun`

**Uses:**
- `plan:internal/runfsm#Asides`
- `plan:internal/runfsm#ReasonOwnedElsewhere`
- `plan:internal/runfsm#ReasonLanded`
- `plan:internal/pairteardown#AskAct`
- `plan:internal/pairteardown#ActRefusal`
- `internal/battencli#ownedElsewhereFor`
- `internal/battencli#OwnedElsewhereError`

**ImpactSummary:** `lym batten run` refuses before its pre-run writes anything when the act rule answers wait or hand_over.

**Intent:**
Make a hand or dobby `lym batten run` ask the act rule first (D1).

`battenCLI` gains `askAct func(aside runfsm.Asides) (pairteardown.ActAnswer, error)`, which `battenCLI.wire` fills over the prime with `pairteardown.AskAct`;
a receiver with no `askAct` checks nothing, as its other seams do.
`refuseByAct` asks with the shed pause set aside, so `lym batten run` still resumes a run its shed paused.
- A hand-over refuses through `ownedElsewhereFor` with the owner and adopt subject, so the pass exits with the owned-elsewhere code dobby reads.
- A landed verdict passes: a done record goes on to the start guard, which clears a done motif run or refuses with its own way forward.
- Every other wait refuses through `pairteardown.ActRefusal` with the run's re-run as the step that follows: a pause intent, a cancelling or cancelled run, or a pause the pause record reached after the covered check.
A failed ask refuses naming what it could not read and the re-run.
`battenPreRun` calls it after the covered and continue-pass checks and before it reads or writes the run's status.

`TestRefuseByAct` (tier 0, a fake `askAct`) reaches each branch: a hand-over returning an `OwnedElsewhereError`, a pause intent naming `lym tender resume <slug>`, a cancelling run naming the cancel, a cancelled run naming `--remove`, a landed and an act verdict passing, and the shed pause set aside in the ask.

**Commit:** 12: lym batten run asks the act rule before its pre-run
