```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out as proposed, looking for a case where the proposed change itself is wrong.

Proposals checked: F1, F2, F3, F4, F5, F6, F7, F8, F9, F12, F13, F15, F16, F17, F18. F14 was fixed with the fixer's own change, so it is out of this lens.

1. MEDIUM — D12, "records one directly from disk only when the attach answers not found" (F1).
   A crash after the attached or original Wait returns with gates still failing, but before state records the outcome, leaves no run to attach to.
   Shuttle's finalize removes the run directory once the run reaches `OutcomeDone`. The comment at `internal/shuttleengine/attach.go:525` says "finalize already removed that run's directory".
   The attach then answers not found, and Weaver records the report from disk without the gate verdict.
   The batch can be recorded done with failing gates, which is the outcome F1 set out to prevent.
   Recording from disk needs its own rule: either rerun the card gates and treat a row gates entry as unverified, or count it as a failed attempt.

2. MEDIUM — D7, "A stopped batch whose own cards changed refuses, naming `lym weaver reset` as the way forward" (F9).
   The natural answer to a `NEEDS_PARENT` question about an ambiguous card is to edit that card.
   An arbiter fix for a batch that failed because of a plan error also edits the stopped batch's own cards.
   Both cases now refuse, and the only way forward throws away every done batch of the run.
   Webster handles the same case per card with `lym webster rebaseline --card NN` (`internal/websterengine/partition.go:131`).
   The proposal also conflicts with D7's own new `NEEDS_PARENT` path, which assumes the run continues after the parent answers.

3. MEDIUM — D6, "State first records the attempt's outcome and failure cause (D9), and then the strand is stopped", combined with "a fault in Weaver's own config or state" (F3 plus F13).
   When the fault is in Weaver's state, for example a state save that fails or a state file that fails to load, the first step cannot happen.
   As written, the strand is then never stopped, so it stays live beside the tackler or arbiter: the case F3 was meant to close.
   The record needs a rule for when the state write fails, such as stopping the strand anyway and naming the fault.

4. MEDIUM — D7 bound, "records the batch done only when its post-batch checks, its card gates and the row's `gates` entries all pass" (F7).
   Go never runs a row gates entry, so the only check behind "all pass" is that the last attempt's recorded failure was not a gates entry.
   Concrete case: the last attempt fails on a card gate, then the arbiter's fix commit breaks a row gates entry such as a lint.
   Re-entry passes the post-batch checks and card gates and records the batch done, though no gates entry ran on the fix.
   The sentence claims a pass that nothing checks.

5. LOW — D2, the reset's in-flight refusal names re-stepping the row "under `lym loom pause --after` the implementer row" (F6).
   `pause --after` takes effect only when the row ends, and the Weaver row ends only after every remaining batch and the verify.
   An operator who wanted to reset must therefore let the whole run build first.
   The reset stops no strand itself, and no way forward ends just the live strand, for example through shuttle's stop verb.

6. LOW — D2 and D3, the reset restores the plan directory to "the weft commit the first begin records" (F5).
   A Plan-Review reroute after the first begin may have approved changes to pending cards.
   The reset silently reverts those approved changes along with the rename rewrites.
   The record does not say whether that is intended.

Tool calls:
- grep of the shuttle run outcomes and `isTerminalOutcome` in `internal/shuttleengine`: my context lacked which outcomes are terminal and whether a finished run stays attachable, which finding 1 depends on.
