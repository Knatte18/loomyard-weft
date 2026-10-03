# Fixer report — plan, round 1

## Fixed

- **F3:** the overview's "step error envelope stays extras-free" decision is replaced by "a failed halt's error envelope carries the friction status".
  Card 2 now puts `AfterStep`'s return on the error envelope of a `shed.Step` error, adds `friction` to every step error envelope's closed key set (empty on `PreStep`/`BuildShed` refusals), has the generic `run` body merge `PostRun` extras onto the error envelope, and has `battenPostRun` return nil on a non-nil `runErr` so batten's error envelope is unchanged; it gains `run.go`, `run_test.go`, `internal/battencli/arm.go` and a new `internal/battencli/postrun_test.go`.
  Card 6's `failed` arms return their status instead of discarding it, with a step error-envelope scenario.
  The verify section adds `./internal/battencli/...`, and card 11's step-envelope doc line covers both envelopes.
- **F4:** card 6 gains a "What this lets through, and the bound" paragraph: halt-time filing under both verbs, the `friction_timeout_min` delay, the held-lock skip, `lyx selfreport create` to `Knatte18/loomyard` only, and card 10's content rule.
- **F5:** card 1 bounds one `Reflect` call to one `deps.Timeout` measured through `deps.Clock`; a spawn after an attach wait gets the remaining budget, and with none left spawns nothing and returns `failed`; a test covers both cases.
- **F6:** card 1 states that record read and write failures and a settle-path spec-build failure return `StatusFailed` with a nil error, and that an unparsable record is discarded and its notes reflected again; tests cover the unparsable and unwritable cases.
- **F7:** card 6 tests a `done` result reporting the row's `failed` status, states that the scenario's fake shuttle stands in for the agent's filing with the `selfreport` knob off, and card 7 reruns the scenario with a fake Tier 1 filer on `c.fileIssue`.
- **F8:** card 9's "the stop report of every stop records the envelope's friction value" sentence is removed.

`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` findings only.

## Deferred

- **F1 (card 7, Tier 1 under step) and F2 (card 8, pending-retry):** not fixed, because fixing them means choosing between the operator's recorded instruction and the decision record, which is an operator decision.
  The board entry `agent-self-reflect` records Tier 1 under `step` as added by the operator after Discussion on 2026-10-03, knowing the record kept it `run`-only, and carries the resumed-halt check card 8 answers.
  Deleting the cards would silently drop that operator scope; keeping them leaves the plan outside its answer key.
  Resolving it needs either the decision record amended through a `goto` to Discussion, which is outside this round's write surface, or the operator dropping the addition, after which cards 7 and 8 and their dependent parts (the overview's operator-additions, finding and "who files a halt" sections, card 6's `tierOneOn` ownership line, card 10's halt rule, card 11's Tier 1 clauses) come out.
