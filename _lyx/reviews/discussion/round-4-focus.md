---
round: 4
focus:
  - "Teardown and loop death: when the detached loop is killed at session end or crashes on its own, does the decision record state a mechanism that stops the whole process tree of the in-flight step, not only the direct child, on Linux, given that the parent-death signal reaches only the direct child and the group kill gated on the child's start time runs after the child is gone? Settle whether the teardown and loop-killed acceptance rows assert that a grandchild of the step is dead."
  - "Windows kill identity: the decision record gates the loop kill on a matching process start time, and the repo's Windows start-time read always reports unreadable. Settle which identity check or kill route the record names for Windows, and what bounds it against a reused pid."
  - "Windows detach: settle whether the record states which detach form the loop uses on Windows and why the loop survives the driver's shell if that shell sits in a job object."
  - "Envelope currency: settle whether the currency check changes when a stopped run is resumed with the same current producer and history length, and whether the record states that a waiter retires the envelope it prints."
  - "Dead-loop stop when another holder has the run lock: settle whether the record states if the pid file is kept and an envelope written, so the next invocation retries instead of spawning a loop over a half-run producer; and whether a waiter whose spawned loop lost the loop-lock race can be mistaken for a dead loop."
---

Focus targets the open gating keys; each entry restates the question without relying on earlier rounds.
