# Card 10 — batten-run-act

**Create:**
- `plan:internal/battencli#battenCLI.refuseByAct` -> `func (c *battenCLI) refuseByAct() error`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#battenCLI.startFirstPass`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`
- `internal/battencli#TestBattenPreStep_RefusalKinds`
- `internal/battencli#TestStartFirstPass`

**Uses:**
- `plan:internal/pairteardown#ReadRun`
- `internal/runfsm#Judge`
- `internal/runfsm#Verdict.Refusal`
- `internal/battencli#newFakeReceiver`
- `internal/battencli#TestBattenPreRun_ContinuePass`

**Intent:** `lym batten run`, `lym batten step` and the first pass that adopting verbs start ask the act rule before they drive anything (D1).
`battenCLI` gains `runVerdict func() (runfsm.Verdict, error)`; `armAt` fills it with `pairteardown.ReadRun` and `runfsm.Judge` with `ShedPause` set aside, since these verbs resume batten's own shed pause.
A nil `runVerdict`, which only a hand-populated receiver has, checks nothing.

`refuseByAct` returns nil on act and otherwise `Verdict.Refusal(slug, "run \"lym batten run <slug>\" again")`, wrapped so the pass's exit code keeps its meaning:
a pause-covers wait in `coveredRefusal` (exit `PassExitCovered`), a hand-over in an `OwnedElsewhereError` carrying the owner, subject and the refusal text (exit `PassExitOwnedElsewhere`), any other wait plain (exit 1).
A terminal wait on `Landed` with a batten record returns `doneSlugRefusal`, which names the cleanup and the leftover branch to delete, the steps a re-run of a done slug needs.
A read error refuses naming the read and the re-run.
- `battenPreRun` calls it where it calls `refuseCovered` today, for every pass but a continue pass, which the dobby gives only after asking the rule itself (card 11).
- `battenPreStepUnmarked` calls it after its run-lock probe and returns a wait as `shedverbs.KindBootstrap`, the kind of every other pre-producer refusal there.
- `startFirstPass` calls it in place of `refuseCovered`, before it spawns anything.
`lym batten goto` does not ask: it moves a halted run's row back and leaves it paused, driving nothing, and the run's next driving verb asks.

A batten-paused run now waits like any paused run: the act rule's order puts `paused` before `landing`, so the dobby no longer finishes a paused run whose work reached its parent.
`battenVerbTexts`' Pause and Goto texts say so: a paused run stays held until `lym batten run <slug>` resumes it.

Tests, tier0:
- `TestBattenRun_RefusesWhileAPauseCoversTheSlug` sets `runVerdict` on its `newFakeReceiver` to `runfsm.Judge` over the records it seeds, with `PauseCovers` read through `pauserecord.ReadCoverage` from the pause record each row writes, so its covering rows refuse through the act rule with the covered exit code and the tender-resume way forward.
  It gains rows over a fake `runVerdict`: act (the pass goes on to its status check), a hand-over (owned-elsewhere exit code naming the owner), a terminal cancelled wait naming `--remove`, a terminal landed wait with the done-slug refusal, and a read error.
- `TestBattenPreStep_RefusalKinds` gains a wait row returning `KindBootstrap` with the refusal text.
- `TestStartFirstPass` gains a wait row that refuses with no pass spawned.

**ImpactSummary:** One new check in batten's run, step and first-pass entries replaces the pause check there, with exit codes unchanged; the pause and goto help texts change.
