---
round: 1
focus:
  - "internal/reportcli/timing.go: does the fallback trigger on every unreadable-journal case (journalctl absent, failing, unparseable) and never on a readable journal that records no sleep? Check the new third result of sleepjournal.Read at each call site."
  - "internal/reportcli/timing.go: are hub-scope pause intervals built correctly (resumed: pause to resume; live: pause to now), with pair-scope entries excluded, and are read errors of the pause record surfaced in sleep_note rather than swallowed?"
  - "sleep_note wording in timing.go against internal/reportcli/doc.go and the verb's help text: does every branch of the note say the figures may include sleep, and do the docs describe the same branches?"
  - "tools/tokencount/sleep.go and internal/sleepjournal: does the signature change leave every caller correct, with tokencount output unchanged?"
  - "internal/reportcli/cli_test.go TestTimingSleepNote: does each new row cover a behavior the diff adds without duplicating another row, and is any added behavior left uncovered?"
---

Seed focus from the summary and the board entry: the entry asks for another idle source when the journal is unreadable, plus a stated note.
