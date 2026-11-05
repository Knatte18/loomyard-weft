# Loop exited without an envelope while step Webster was in flight

`loop-exit-no-envelope-cause` · Task · bug, shed, friction · running · Drift-Verify

Imported from [issue #51](https://github.com/Knatte18/loomyard/issues/51).

What happened: the run halted as failed at step Webster with anomaly interrupted-stop. The status error was `loop-exited: loop ... ended without an envelope; its output is in .lym/shed/<task>/steps/loop.log; step Webster was in flight under trace ...`, followed by `shedverbs: step Webster interrupted (loop-exited)`.
The way forward named was to read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, and re-run `lym shed step <task> --until-stop`.

Why it is abnormal: a loop must end by writing an envelope. The note records no cause for the missing envelope (no crash, signal or budget reason), and the preceding history shows all earlier steps through Drift-Await finishing done, so the exit came while Webster was starting. The halt note carries nothing that lets the operator tell a loop crash from an external kill.

What to do: have the loop-exited anomaly record the exit cause (exit code or signal, last lines of loop.stderr_path) in the halt note itself. Also check whether a loop can exit between Drift-Await done and the Webster start without writing an envelope, and make that path write one.

Provenance: task server-based-live-viewer; notes loom-halt-2.md.
