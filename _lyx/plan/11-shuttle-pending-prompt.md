# Card 11 — shuttle-pending-prompt

**Create:**
- `internal/shuttleengine#Run.DeliverPendingPrompt`
- `internal/shuttleengine#TestRun_DeliverPendingPrompt_SendsOnlyAPromptWhoseTurnNeverStarted`
- `internal/burlerengine#promptDeliverer`

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Runner.start`
- `internal/burlerengine#Engine.ResumeFix`
- `internal/burlerengine#fakeHandle`
- `internal/burlerengine#TestEngine_ProbeFix_And_ResumeFix`
- `internal/shuttleengine/doc.go`
- `internal/burlerengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Make burler's live-fix resume (`burler: resuming a live fix`) attach only to a strand whose task prompt's turn started, and deliver the prompt first when only the skill-load preamble did (D9).
A fix chair whose step died between its skill-load turns and its prompt delivery is live and attachable today, and the resumed fix then waits on an agent that never got its task.

- `RunState` gains `PromptLine`, JSON `promptLine`, omitted when empty: `start` persists the launch's deferred prompt line in the record it first writes, so a later attach can deliver it. A launch whose prompt is on its command line has none.
- `Run.DeliverPendingPrompt` reads the persisted record: with no `PromptLine` it reports false.
  Otherwise the prompt's turn started when a turn start lies at or past `PromptOffset` in the events file (`turnStartedSince`), and then it reports false.
  When none does, it persists `PromptOffset` as the events file's current size when it was zero, sends `PromptLine` through the verified send path (`sendVerified`, the guarded paste-submit, whose pending-send record already keeps a text left in the box from going twice), and reports true.
  A send error is returned and nothing else changes.
- `burlerengine` declares the optional capability `promptDeliverer` (`DeliverPendingPrompt() (bool, error)`); `ResumeFix` calls it on `live.Handle` when the handle has it, before `e.seats.Resume`, logs `burler: delivered the fix's task prompt before resuming` when it sent, and returns a delivery error wrapped as the resume's error so the row reruns the fix fresh.
  `*shuttleengine.Run` has it; a seat fake without it resumes as today.

D10's stop of a killed step's runs (card 20) means the strand left by a watchdog kill is gone before the relaunch, so this reattach applies to a strand left by another death.

Tests: the new shuttle test reconstructs runs from records over the fake reed: a record with no prompt line reports false and sends nothing; a record whose events hold a turn start past `PromptOffset` reports false; a record whose events hold only the load turns sends the prompt line once and reports true; a record with a zero `PromptOffset` persists the offset before sending.
`TestEngine_ProbeFix_And_ResumeFix` gains a row whose live fixer's `fakeHandle` reports a pending prompt: `ResumeFix` calls `DeliverPendingPrompt` once before the seats resume and still returns the done fix; a row whose delivery fails returns that error and resumes nothing. `fakeHandle` gains the method, answering from two new fields.
Both package docs name the delivery; the shuttleengine budget row grows by one.

**ImpactSummary:** RunState gains an optional field, Run a delivery method and burler's resume one guarded call; runs launched without skills and seat fakes without the method are unchanged.
