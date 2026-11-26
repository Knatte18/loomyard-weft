---
round: 1
focus:
  - "internal/shellguard doc.go and rows.go: check each new claim about how an exception is spelled and what the scan stops at, matches at and refuses against the body of unquotedWordOtherThan, sessionBusCommandPattern and the dbus-send form, including quoted, escaped, longer-flag and wrapped-shell inputs; is any stated guarantee, fail-closed included, broader than the code gives?"
  - "internal/shellguard documentation as guidance for the next row author: would following only the package doc, the Row.Patterns comment and the helper comment produce a fail-closed exception for a command other than dbus-send, and does each place agree with the others and with the amended board entry's Part 2?"
  - "internal/shellguard rows_integration_test.go: does the added session-bus case pin the later-command property it names, in the right direction of the table's expectation column, and does an existing case already cover it?"
  - "internal/shuttleengine/claudeengine settings_test.go: does -update write every input the golden is meant to pin, reproduce the committed file, and work through the documented lym gate test command, and does the compare path still fail on every difference it failed on at the review base?"
  - "_lym/landing/summary.md: check its claims against the diff, including the byte-for-byte statement and the Merge danger section's door and blast radius."
---

Round 1 has the whole committed range to cover; these entries mark where to look first and do not limit it.
