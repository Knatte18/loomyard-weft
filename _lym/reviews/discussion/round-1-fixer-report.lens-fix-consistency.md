```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checking whether each part the fix changed still agrees with the sections it touches, and whether any change is missing its twin elsewhere.

**MEDIUM**: the Constraints bullet "every `verifyrun.Run` caller removes `agentname.StrandNameEnv` … card-verify by D2"
- The Constraint says card-verify is covered by D2, but D2 only strips the strand for the `lym gate card-verify` verb ("`lym gate card-verify` removes `agentname.StrandNameEnv` from the environment of the line it runs").
- `cardverify.Runner`, the code that calls `verifyrun.Run`, is also run from Go verbs (D7: "through `internal/cardverify` from a Go verb"). With `Slots` nil it runs every line "unslotted with the parent's environment" (`internal/cardverify/runner.go`, the `Runner.Slots` doc and `runLine`).
- So under a strand-carrying process, a non-`--wait` line run by a Go verb keeps the strand and has no inherited slot. That is the detach the Constraint says cannot happen.
- Fix: the twin should be `cardverify.Runner` itself stripping the strand, or D2 should say so.

**LOW**: D3's new Bound "until the latest deadline a front wrote, at most the long cap (D5) plus the grace" against D3's Rationale, which the fix left as it was
- The Rationale still says the slots "free within one cap plus the grace". Under the new rule a long-cap front's deadline holds the run, so "one cap" now has to mean the long cap. The Rationale's twin was not updated.

**LOW**: the PATTERN-tolerant-records Constraint, given the new field `Abandoned` in D5
- The Constraint says only "the call id is optional on read, and records without it still total in `lym gate report`".
- D5 now adds a second new `RunRecord` field, `Abandoned`. The tolerant-records line and the D5 acceptance criterion ("records without a call id still total") still name only the call id.

**LOW**: D6's first bullet, "A `--wait` gate line, which D7 keeps for every card"
- It reads as if every card's line is a `--wait` gate line. That disagrees with the second bullet's "Any other line takes one slot inside the runner".
- It also disagrees with D7, which keeps `--wait` only on a card line that is a gate call ("a card's `**Verify:**` line keeps its required `--wait`"). `IsGateCall` accepts only `lym gate test … --wait`.

**LOW**: the D1 text "The front that starts a run holds the key directory's lock until the runner's process file exists", against the PATTERN-daemon-wakeups Constraint
- The Constraint lists only "the front's wait and the runner's lease read" as waits that wake at most once a second.
- The fix adds a third wait, the starting front polling for the process file under the lock. Its wake rate and its bound are not stated, and the Constraint was not extended to cover it.

I made no further tool calls; my context held everything this lens needed.
