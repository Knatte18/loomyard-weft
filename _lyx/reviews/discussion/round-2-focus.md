---
round: 2
focus:
  - "In decision-record.md D7, the watchdog is skipped for a `lyx gate test` running in an inherited gate slot (`LYX_GATE_SLOT`). `verifytree.Verify` also acquires a slot and passes `LYX_GATE_SLOT` to the hub's verify command, which runs `lyx gate test` itself. Settle whether the exemption still lets the watchdog run on that path, and what bounds it where it is skipped."
  - "In D7, the busy threshold is at least one full core per sample. Settle whether a single spinning goroutine, with clock-tick granularity in `/proc/<pid>/stat` and other packages running beside it under the slot's `-p` cap, reliably clears the threshold, and whether the integration acceptance criterion rests on the same boundary."
  - "In D6 and the refusal-spec row it changes, a hang quit by the watchdog on a healthy CPU-bound package ends stuck. Settle whether the stuck reason, report and findings name a way forward that fits that case, as `PATTERN-refusal-way-forward` requires, and whether an acceptance criterion pins it."
  - "In D5, an exit code of 128 or above counts as a hang, and the verify command is any hub-configured shell command. Settle whether D5's Bound and D6's wording stay true for a non-signal exit such as git's 128 or a tool's 255."
---
