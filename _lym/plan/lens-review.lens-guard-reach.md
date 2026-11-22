```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review: for each guard, refusal, branch and test row the cards add, can a real entry point reach it, and does the guarded site recur unguarded?

**BLOCKING — Card 10 (D4): the first session of an autonomous review starts interactive.**
- `PRReview.ensureSession` builds the spec before it writes anything: `p.sessionSpec(rec)` runs at `internal/landingshed/prsession.go:76` (function line 19).
- Card 10 records `rec.Autonomy` only "in the same `updateRecord` that writes the session fields" (`prsession.go:92-93`, after the start).
- On a record holding no autonomy, `sessionSpec` therefore sees nil, reads it as not true, and sets `Interactive` true with the operator text.
- The planned row "autonomy true on a record holding none: the started spec has `Interactive` false" cannot pass as specified.
- Fix: set `rec.Autonomy` from `Autonomy()` before `sessionSpec(rec)`, and persist it with the session fields or before the start.

**MEDIUM — Card 6 (D3): the cleared-ask skip lets rulings stand with no parent agreement.**
- The halt fires only when the spawn's latest ask is "not cleared" and ended.
- After the resume, `resumeJudgeSession` clears the ask, and `runAgent` (`internal/driftengine/verify.go:883-923`) runs `agent.Call` again through `NewMultiLLMProducerSourced`.
- Nothing in the card shows that a judge already ended Done re-runs its gate entries on that call. If it does not, no round-2 ask is recorded, the latest ask is the cleared one, the halt check skips it, and `storeRulings` stores the rulings.
- That is the outcome D3 forbids under autonomy.
- The planned row "a second call after the halt leaves the ask `Cleared`" uses a fake judge that bypasses the gate, so it cannot catch this.
- Fix: treat a latest ask that is still cleared after `runAgent` as not agreed (halt again). Add a row asserting that a new ask is recorded on the resume, or that the call halts.

**MEDIUM — Card 14 (D4): the goto clear has no guard, and the card's bound rests on an unverified claim.**
- D4 says goto onto PR-Review is admitted "only on a run halted at PR-Review or later, never on one PR-Review still holds."
- No card adds that refusal. `prReviewResettingGuard` clears whenever the inner guard admits.
- The inner guard is `settledGuard`, which composes only `finalizeGuard` and `settledSegmentGuard` (`internal/loomcli/settledguard.go:21-29`). Neither checks the run's current row.
- So a goto onto PR-Review from a run halted at an earlier row is admitted and clears the record.
- The card's bound, "Goto moves only a run whose driver is not live", names no check: `internal/shedengine/goto.go` has no live or driver test.
- No planned row reaches either the row condition or the liveness condition.

**LOW — Card 14: one guard site stays outside the wrapper.**
- `internal/loomcli/arbiter.go:159` passes `c.settledGuard()` bare as `shedarbiter.Deps.GotoGuard`, not through `gotoGuardEndingFixers`.
- The card says every site builds its guard through the wrapper.
- The arbiter's consume path (`arbiterconsume.go:29`) is wrapped, so this matters only if the arbiter's own `GotoGuard` moves a run. That is unconfirmed.

Calls made beyond my context:
1. grep of `gotoGuardEndingFixers` and `settledGuard` callers, plus `fixerendguard.go:40-80`: to trace which goto sites card 14's wrapper covers.
2. `arbiter.go:145-170`, `settledguard.go:15-45` and `ReadBeamLandBlockers`: to confirm what the inner goto guard checks and whether card 15's `WorkRemains` counts drafts (it does: every non-done task).
3. `beamland.go:111-150`, the liveness grep in `goto.go` and `slackcli` `wireProduction`: to check the goto-liveness claim and that the hook has the orch's location.
4. `verify.go:453-530` and `resumeJudgeSession`: to trace card 6's resume path.
5. `prreview.go:325-347` and `prsession.go:58-102`: to check the order of `sessionSpec` against the autonomy write on card 10's path.
6. `verify.go:878-925` (`runAgent`): to see whether a resumed judge re-runs the agreement gate.
