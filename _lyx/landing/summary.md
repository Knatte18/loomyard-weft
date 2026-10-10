# Webster audit and recovery: pathless findings, read-only classifier

Closes six gaps in the webster audit and recovery paths that a `cli-surface` run hit.

## What changes

- **`recover-batch` and pathless findings.**
  An uncheckable entry recorded for a correctness finding with no path (the `<class>: <detail>` form `failOnCorrectness` writes) is now recognised by one predicate beside that writer.
  `contractWritesFor`, `splitContractPaths` and `uncheckableReason` skip such an entry, so it never reaches `canonicalPath`.
  A long shell command no longer fails `filepath.EvalSymlinks` with `ENAMETOOLONG`, and `recover-batch` returns its documented refusal naming `accept-audit --batch NN`.
  Path entries still resolve and still error on a genuine link failure.
- **Read-only classifier (`IsReadOnlyCommand`).**
  `lyx`, `git` and `sed` segments are split into words honouring quotes, so `git log --format='%h %s'` is read-only.
  A dequoted word starting with `-` is still a flag and must match as before, so a quoted writing flag is rejected.
  Unquoted globs are admitted only after `--` in `git log`, `show`, `diff` and `status`; quoted globs are text.
  A leading `~` is admitted, and so are `2>&1` and `2>/dev/null` in any segment.
  `sed -n <N>p` and `sed -n <N>,<M>p` on plain file words join the readers.
  A trailing `--json` is a help form (`lyx fabric add --json`, `lyx fabric --help --json`).
  Loops, `$` expansions, substitutions, backslashes, other redirections and `bash -c` stay non-read-only.
- **Git reader flags.**
  The admitted flags of each git reader form were reviewed against git's documentation, and the classifier table now pins one writing flag rejected per form (`show --output`, `diff --output`, `remote set-url`, `branch -m`, `config --file`).
- **Merriam-base failure text.**
  `MerriamBase`'s error states the file and cause only, and each caller words its own way forward.
  The auto-rebaseline refusal now holds exactly one `way forward:` clause, the run's re-entry step, instead of also naming a verb re-run that the Webster row must never name.
- **`begin-batch` existing-report remedy.**
  The remedy follows `recover-batch`'s own order.
  A record at the recovery cap with no unrendered amendment wraps `ErrRecoveryExhausted`, names the batch's recoveries as exhausted and gives the reset routes `recover-batch` already names.
  The `begin-batch` verb sets `recovery_exhausted: true` on that refusal, and Master's begin-batch rung gains one sentence for it.
  A dead recovery follows `RecoveryRetry`, and a stuck or failed record below the cap still names `recover-batch NN`.
- **Thin rows.**
  Rows were added to the rebaseline table ("accepting no card file", "amended in-flight cards") and to the shell-wait table (transcript- and payload-reported shells land in `EndedShells`, an awaited shell does not).

`contracts/specs/refusal-spec.md`, the `IsReadOnlyCommand` doc comment and `internal/websterengine/doc.go` carry the new rules.

Closes #513
Closes #514
Closes #518

## Design decisions

- The pathless predicate keys on a known audit class followed by `: `, so every pathless class shares the rule and `allPathlessFabricReference` keeps its fabric-only meaning for the accept route.
- Zsh `EXTENDED_GLOB` reads a leading `^` and an `x#` as globs, so the classifier treats both as glob characters.
  A revision exclusion must be written quoted (`'^main'`) or as a range.
- `sed` is admitted in the `-n` line-print form only, because GNU `sed` reads flags after file operands and a script other than a line print can write (`w`) or execute (`e`).
- A command declaring its own `--json` shadows the global flag and runs, so the classifier cannot treat a trailing `--json` as help for it.
  A test in `cmd/lyx` walks the tree `newRoot` builds and pins the set of commands declaring their own `--json`.
  The classifier does not know that set; a new entry is a deliberate, reviewed change.
- The `begin-batch` exhausted case carries the flag at any record status, so `begin-batch` and Master never disagree on one state.

## To check by hand

- The `--json` pin lists `batten status`, `loom status` and `shed status`, not `shed status` alone as the design assumed.
  Confirm each of the three reads only.
- The classifier assumes bash or zsh and sees static shape only.
  Review the unquoted-glob rule after `--` (`log`, `show`, `diff`, `status`), and that `ls-remote` and `remote` reject a glob there.
- A verb gaining its own `--json` later would run under a command the classifier admits; the `cmd/lyx` pin is the only guard.
- The `cli-surface` run adds a command-tree form test in `cmd/lyx`; a conflict near the `--json` pin is resolved at landing.
- `lyx gate test --tags integration ./...` was not part of this change's recorded runs.
