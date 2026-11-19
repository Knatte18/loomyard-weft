---
dispositions:
  - id: F1
    disposition: deferred
    reason: The writer was not found; earlier searches of the suspect packages and the code their tests drive found no path to the shared temp directories, so the instrumentation stays and the parent records the deferral with lym loom decision add.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: deferred
    reason: The proposed drain is a no-op because the consumer goroutine already reads every delivered event; the loss sits inside fswatch.Watcher.Close, which returns on done over a pending event, and a delivery barrier there changes the semantics for every fswatch user.
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changed

Commits d745ce69 and ed7d8b81, one commit for the findings together because their changes share `gitentry.go` and its tests.
`_lym/landing/summary.md` was edited in the working tree and not committed.

- F2: `checkNewGitEntries` sets code 1 only for a sighting that is attributed and has `ownInDir`, meaning this binary or a descendant worked in the entry's directory.
  Every other sighting is a machine finding that keeps the code.
  `snapshotProcesses` returns `attributed` true only when it read this binary's own argv from the table.
  The package doc, the `gitEntrySighting` field docs, the `.git` bullet of `pattern/PATTERN-test-isolation.md` and the landing summary state the new rule.
  `TestCheckNewGitEntries` and `TestRunCheckingGitEntries` hold the new rows, including a sighting with no other test binary and no own process that keeps the code.
- F3: `//testtiming:keep` above `TestClassifyProcesses`, now table-driven.
- F4: the finding text says the writer is one of the listed binaries, this package or a process that is no test binary; the pattern and the landing summary say the named binaries are candidates.
- F6: `watchGitEntries` keeps `map[string][]gitEntrySighting`, and every sighting is reported; a row with two sightings asserts both.
- F7: rows for a watched directory that does not exist, for a move of the entry before a new create, for a looping and a missing parent chain, for an argv over `maxArgvChars`, and an off-Linux test file for the unattributed snapshot.
- F8: `quoteArgv` cuts at a rune; a row with a multibyte argv fails with a byte cut.
- The git-entry tests moved from `tmuxkit_test.go` into `gitentry_test.go`.
- The second commit carries the `Loosened-Check:` trailers for the reworked rows.

## Gate repair

The gate found comment lines wrapped mid-sentence or at 120 characters in `gitentry.go` and `tmuxkit.go`.
A third commit breaks each at a sentence or clause boundary and changes no code.
A fourth commit splits the "Bound:" comment in `tmuxkit.go` into one sentence per line, since the third left it wrapped mid-sentence.

## Disputed

Nothing.

## Deferred

- F1: not found in this run.
  The parent should record the deferral with `lym loom decision add`; the fixer cannot.
- F5: needs a change to `fswatch.Watcher.Close` that drains pending events to a consumer still reading, which affects every user of fswatch.
  The row `RunMovingTheEntryAwayAndCreatingAnotherStillFails` pins the create-only filter only when the create event is delivered before the watch stops, for the same reason.

## Self-check

- fix-code-correctness, MEDIUM, rename row can pass with the filter removed when the create event is dropped at stop: deferred with F5, row renamed to claim only what it holds.
- fix-code-correctness, LOW, unattributed finding text wrong on Linux: fixed.
- fix-consistency, MEDIUM, summary Tests section on the old rule: fixed.
- fix-consistency, MEDIUM, no `Loosened-Check:` trailer: fixed in the second commit.
- fix-consistency, LOW, header line over the limit: fixed.
- fix-proposal, MEDIUM, "so this package wrote it" states more than the evidence: fixed, now "so it is blamed on this package".
- fix-proposal, MEDIUM, own pid counted from the listing rather than a readable entry: fixed.
- fix-proposal, LOW, others list may suggest blame logic: fixed differently, none made; the package doc and pattern say the binaries are candidates.
- fix-proposal, LOW, rows F7 assertions: no defect found, none made.
- fix-affected-set, LOW, summary Tests section stale: fixed.

## Rulings

None.
