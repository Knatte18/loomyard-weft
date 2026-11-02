```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. Proposals the fixer carried out as written: F2, F3 (the `runproc.EndRole` option), F4, F5 (the refuse-at-load option), F6, F7 (the no-parent option) and F8. I did not challenge F1, because the fixer solved it with its own `Final`/`MayHold` approach.

**MEDIUM: D8 bullet 1 and the new D8 criterion get one case of the visit key wrong.** The changed text says a resume of a halted run always starts an empty set. That is false for a run that halted in the failed state without writing a history entry.
- Changed text in D8: "a goto, a resume of a halted run and every fresh row entry start an empty set". The criterion repeats it: "a goto, a resume of a halted run and a fresh row entry start an empty one".
- A failed halt can write no history entry. `persistMarked` (`internal/shedengine/run.go:853`, `:873-898`) states that "a write that appends none carries the label through".
- Resume can follow such a halt. `resumableHalt` covers blocked, failed and awaiting states (`run.go:883`), and on resume the caller label waits for the next appended entry (`PendingBy`).
- Case: a row's gate closure returns an infrastructure error. The step writes `StateFailed` with no new history entry, and the operator resumes. The re-called row reads the same history count and newest `Repeats`, so it gets the same visit key. It reads the earlier call's record set instead of an empty one.
- Effect: the key does what D2 needs here, since it attaches to a still-live fixer. Only the sentence and the criterion promise the wrong thing. They should say a resume starts an empty set only after a halt that wrote a history entry, and that otherwise it keeps the set, as a crash re-step does.

**MEDIUM: the F6 proposal ("no pair slug, no hand-off") contradicts D7 bullet 2.**
- Changed text in D4 Bound: "a call with no such slug, on the prime or outside a hub, hands off nothing, and its red verdict ends the run as today". D1 now says the same.
- D7 bullet 2 renders the fork stencil from the `card` key's `fixer` value alone. With the value set, the fork is told "a failed card gate call has already been through a cold fixer whose reply its report carries".
- Case: a Webster fork whose call has no pair slug, with `card.fixer` at its default `sonnet[medium]`. The red call hands off nothing, yet the fork's prompt says a fixer already ran and that its reply is in the report. No such reply exists.
- D7's Plan-Write and Discussion-Write switch has the same gap. It keys only on the `fixer` setting being non-empty. With no pair slug, the writer is told to stop running its validator loop while no fixer will ever run.
- D7 should also test the no-pair-slug condition. Alternatively, the no-slug case could spawn the fixer unregistered rather than skip it.

F3, F4, F5, F7 and F8: I built no case in which the proposal is wrong. For F3 I checked that `EndRole` on the pair's `gate-fixer` role spares webster's `verify-fixer`, because that registers under its own role, and that it stops a card fixer, which D8 says on purpose.

Tool calls:
- Read `internal/shedengine/run.go:850-909`. My context had only grep lines from this file, not the code that decides whether a failed or resumed write appends a history entry.
