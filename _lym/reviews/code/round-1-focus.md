---
round: 1
focus:
  - "internal/fabriccli/hubmutation_integration_test.go, commit b1fe0b2d: does the new serial test read the process-wide trace sink without racing the parallel anchor tests, and does it restore the sink and any other global it repoints?"
  - "The same file: does moving the sink check out of the scenario step leave the bypass push's other checks in place, and does the new test duplicate coverage an existing test already gives?"
  - "Scope: the amendment is a test-only change in internal/fabriccli, outside the board entry's scope of the scanner, recover-batch, accept-audit and reset. Does it change any non-test code, or touch anything the board entry does not license?"
  - "The Parent-Align merge a7dfe2ea: does anything the parent brought in contradict the settled scanner, recover-batch or reset behavior?"
---

The settled scanner, recover-batch and reset changes are out of this round.
The amendment is the serial-test fix for the trace-sink race and the parent merge that precedes it.
