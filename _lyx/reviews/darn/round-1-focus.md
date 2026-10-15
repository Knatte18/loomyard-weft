---
round: 1
focus:
  - "tools/tokencount `subtractSleeps` and the journal parser: do sleep intervals pair correctly by operation and boot, and does every elapsed and idle figure (run, role, span, idle row, Producers rows) count awake time only, with no double subtraction or negative idle?"
  - "Producers table: is the share active time over summed active time, and is the order active time then name, as the board entry asks?"
  - "`roleOf` `-chair` mapping: does it fold old role names onto the new ones without mis-folding advisor or other roles, and is each role summed once across the rename?"
  - "Scope: is the `-no-sleep` flag and the header and per-run sleep lines licensed by the board entry, and does the missing-journal path degrade to a header line without erroring?"
  - "Summary claims against the diff: the `doc` text in `tools/tokencount/main.go`, the test-budget bump in `cmd/lyx/testdata/test-budget.yaml`, and whether new behaviors (share, ordering, `-chair` mapping, `-no-sleep`) have test coverage."
---

Seed focus only; the reviewer still reads the whole committed diff against the amended board entry.
