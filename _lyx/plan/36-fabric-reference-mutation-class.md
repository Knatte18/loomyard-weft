# Card 36 — fabric-reference-mutation-class

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`

**Uses:**
- `internal/websterengine/recordbatch.go`
- `internal/websterengine/batchfail.go`

**Intent:**
Fix PR #321 finding 2: `ClassifyViolation`'s default classes every fabric-reference finding as policy.
A fork or Master running `git -C <fabric sibling> checkout HEAD~1 -- webster/state.json`, or `rm` on a report, therefore warns and records once, and re-running the cards' verify commands cannot detect a rewrite of run state.
The same write made through Write/Edit resolves through the `_lyx` link and is correctness, so the two routes disagree.

- `AuditViolation` gains `Command string`, the Bash command a fabric-reference finding matched.
  `CheckFork` and `CheckParent` set it on every fabric-reference violation; other classes leave it empty.
- A new unexported `mutatingCommand(cmd string) bool` in `audit.go` reports whether a command can change files.
  It splits the command into segments at `&&`, `||`, `;` and `|`, and is true when any segment:
  - runs `git` (after any `-C <dir>` or `-c <k=v>` options) with a subcommand outside the read-only set `status`, `log`, `show`, `diff`, `rev-parse`, `ls-files`, `ls-tree`, `cat-file`, `grep`, `blame`, `describe`, `for-each-ref`, `show-ref`, `merge-base`;
  - runs `lyx fabric` with a verb outside the read-only set `status`, `diff`, `list`, `pairs`, or `prune`/`cleanup` with `--apply`;
  - runs `rm`, `mv`, `cp`, `touch`, `truncate`, `tee`, `ln`, or `sed`/`perl` with `-i`;
  - carries an output redirect (`>` or `>>`, excluding `2>&1`-style descriptor duplication and `> /dev/null`).
  Every other command is read-only.
- `ClassifyViolation` classes a fabric-reference finding as correctness when `mutatingCommand(v.Command)` is true, and policy otherwise.
  Its doc comment names the rule.
  The Detail of a mutating fabric-reference finding says the command can rewrite run state, so the failed batch's digest and the run-exit stuck reason name it.
- `doc.go`'s audit-findings section adds the mutating fabric reference to the correctness examples and keeps read-only fabric references as policy.

Tests:

- `auditseverity_test.go`: a table over `ClassifyViolation` with fabric-reference findings whose `Command` is `git -C <dir> checkout HEAD~1 -- webster/state.json`, `git -C <dir> reset --hard`, `git -C <dir> stash`, `rm <dir>/webster/reports/01-a.yaml`, `echo x > <dir>/webster/state.json`, `lyx fabric commit`, `cat <dir>/webster/state.json | head` and `git -C <dir> log -1`, `lyx fabric status`; the first six are correctness, the last three policy.
- `recordbatch_test.go`: an OK report on a batch whose fork transcript ran `git -C <fabric dir> checkout HEAD~1 -- webster/state.json` → the batch is recorded failed with the report archived, the digest names the command, and `recover-batch` proceeds; the existing read-only fabric-reference case still records done with one warning.

**ImpactSummary:** Mutating fabric-reference findings become correctness, failing the batch or demoting run exit; read-only ones stay recorded warnings.
