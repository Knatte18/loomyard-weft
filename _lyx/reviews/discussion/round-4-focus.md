---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D6, check where the per-verb engine check runs against `internal/webstercli/wiring.go`, `internal/webstercli/recoverbatch.go`, `internal/loomcli/wiring.go` and `internal/websterengine`: wiring runs once per invocation for every verb, so settle whether the stated check site fires only on a verb or path that actually spawns, and whether an attach, await, terminal re-read, `record-batch`, `reset`, `validate`, or loom `status` and `pause` can be refused by it."
  - "In D6, check that the per-run engine state keys (the master engine, the batch engine and the recovery-session engine map) name their home record, and that every writer that rebuilds a `BatchState` wholesale, `internal/websterengine/beginbatch.go` re-begin and `internal/websterengine/recoverbatch.go` respawn, is told to carry them forward the way it carries `RecoverySessions`; settle whether a second recovery or a re-begin can leave a session read through the current default."
  - "In D6, for every state entry that carries no engine name, and for a recorded engine name the engine set does not hold, settle whether the Decision states the outcome and what bounds it, per the attack-surface question; a bound that holds only while one engine is registered must say when it stops holding."
  - "Settle whether D6's rule sentence and its `Bound:` sentence about which strands each verb's check covers are true for `run`, which checks the recovery strand's engine although only Master's later recover-batch calls spawn it."
---

Focus targets D6 only; the other decisions drew no gating finding this round.
