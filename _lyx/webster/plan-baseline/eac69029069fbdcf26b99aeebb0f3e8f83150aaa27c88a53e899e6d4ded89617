# Card 11 — batten-loop-arming

**Create:**
- `plan:internal/battencli#battenLoopSpec` -> `func battenLoopSpec(location *lyxcwd.Location, slug string) shedverbs.LoopSpec`
- `plan:internal/battencli#battenInterruptPolicy` -> `func battenInterruptPolicy(row string) string`

**Edit:**
- `internal/battencli#battenCLI.specFor`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#TestSpecFor_FillsRunIdentityForSlug`

**Intent:**
Arm batten's `--until-stop` loop and its per-row interrupt policy (D4, D2), so `lyx batten step <slug> --until-stop` runs, the entry the driver body of card 13 steps through.
`lyx batten run` keeps its generic body in this card.

- `battenLoopSpec` builds the `shedverbs.LoopSpec` of batten's own run: the lock, pid file, log, envelope, job name, delivered marker and stop files from `shedrun`'s loop constructors over the prime location and the slug, the trace listing through `logger.TraceFilesFor(logger.TraceDir(), ...)`, and the executable from `os.Executable`, as loom's `loopSpecFor` builds them.
  It leaves `IdleTimeout` zero and `Activity` nil: Run-Shed waits idle by design, so the watchdog stays disarmed (D4).
  A wedged batten step is never killed by the loop, as today's in-process generic run body kills none either.
  It is bounded by the operator's `lyx batten pause <slug>` at the next producer boundary or by ending the detached loop job; the interrupted rule then applies to the killed step under its row's policy below, a re-step under the repair cap at Run-Shed and a hand-back at any other row.
- `battenInterruptPolicy` answers `shedverbs.InterruptPolicyReinvoke` for `battenrecipe.NameRunShed`, whose wait reattaches to the child run, and `shedverbs.InterruptPolicyHandback` for every other row name, the empty one included.
  So an interrupted Run-Shed step re-steps under the repair cap and an interrupted step at any other row hands back (D2).
- `specFor` sets `Hooks.InterruptPolicyFor` to `battenInterruptPolicy` for every verb, so `status` and `step` report the policy, and sets `Loop` from `battenLoopSpec` when the receiver has a location.
  It wires no `ConsumeAttempt` hook, so the loop refuses an arbiter attempt by today's `shedverbs` refusal, and no `AfterInterrupt` hook, since batten keeps no friction directory.
- `battenVerbTexts.Step.Long` says `--until-stop` runs the step loop to the run's next stop, the entry `lyx batten run` drives.

`TestSpecFor_FillsRunIdentityForSlug`, which already arms a hand-populated receiver with a location for each verb, gains assertions and one row:
- the loop's lock, pid, log and envelope paths equal `shedrun`'s constructors for the prime and the slug;
- `IdleTimeout` is zero, and `Activity` and `ConsumeAttempt` are nil;
- the policy hook answers reinvoke for `Run-Shed` and handback for `Land`, `Worktree-Teardown`, `Seed-Child` and the empty name;
- a receiver without a location gets a zero `Loop`.

**ImpactSummary:** batten's spec gains a loop and an interrupt-policy hook; its run, step, status, pause and goto verbs share the spec and change only in what step --until-stop and the envelopes' interrupt policy report.
