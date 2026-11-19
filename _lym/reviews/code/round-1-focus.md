---
round: 1
focus:
  - "internal/shedverbs/loop.go `runLoop`: the issue asks for a recorded reason on every path that ends the loop without an envelope. Settle whether any such path still ends silently, among the early returns, the panic path, and `OnStart` or `OnExit` failures."
  - "internal/shedverbs/loopwait.go `unmarkedExitCause`: settle whether comparing `proc.BootTime` (whole seconds) with `StartedAt` (sub-second wall clock) can misname the cause at the boundary. Check the Windows `BootTime` (sleep counted, clock drift) and a wall-clock step between boot and loop start."
  - "internal/shedverbs/looppid.go and the `LoopPIDRecord` readers: settle whether `StartedAt` with `omitzero` stays compatible with older pid files and every place that copies or rewrites the record (the in-flight child updates), so the field is neither dropped nor zeroed."
  - "internal/shuttleengine/waitshadow.go `loopAgreesWithState`: settle whether counting held beside busy/background as agreement matches the issue's third ask (escalate or self-resolve the repeated warning), and whether it hides a real held/busy disagreement, such as a stuck session."
  - "internal/proc/proc_linux.go `BootTime`: settle the failure paths (missing btime line, parse error) and the Windows and Linux parity of the doc contract."
  - "Scope against the board entry: settle whether the diff covers all three 'What to do' bullets, and whether any change goes beyond them."
---

Seed focus from the diff and the board entry in the decision record, which holds only the Goal.
The test-economy and mutation notes list no candidate that needs a focus entry beyond the questions above.
