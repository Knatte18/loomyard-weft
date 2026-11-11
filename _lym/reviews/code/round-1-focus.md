---
round: 1
focus:
  - "internal/shedverbs/loopkeeper.go and loop.go: does the keeper record the loop's exit code or signal on every path that ends without an envelope, including a kill, a crash and a clean exit, and are failures writing the record handled?"
  - "internal/shedverbs/loopwait.go and step.go: does the dead-loop halt note include the recorded exit status, name the in-flight step's trace in `trace_file` rather than `loop.log`, and include the `loop.stderr_path` content or location, as the board entry asks?"
  - "Board entry's request to check why the loop log lacks Drift-* envelopes and whether a normally ended Friction-Reflect session is misreported as interrupted: does the diff address or license leaving each, per the decision record's Goal and its Added after Discussion entries?"
  - "internal/shedverbs/loop_unix_integration_test.go, loop_integration_test.go and loopwait_test.go: test economy against the coverage of the three tests the redundancy report lists, and whether each added behavior has coverage."
  - "internal/shedverbs/doc.go, internal/loomcli/doc.go and pattern/PATTERN-shed-driver.md: do the doc and PATTERN edits match the code as the diff leaves it, and does _lym/landing/summary.md claim nothing the diff does not bear out?"
  - "internal/battencli/arm.go, internal/loomcli/cli.go, internal/shedcli/cli.go and internal/shedrun/paths.go: is each change licensed by the answer key?"
---

Seed focus set from the diff stat and the decision record's Goal; the reviewer reads the full diff regardless.
