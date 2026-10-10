# Decision record — webster-audit-gaps

## Goal

Close the six webster audit and recovery gaps the board entry lists: a pathless audit finding no longer breaks `recover-batch` (#513), the read-only classifier admits help-only `lyx` calls under `--json` and quoted or globbed reader arguments without admitting expansions or loops (#514), every admitted git reader's flags are bounded against writing, a failed Merriam-base read under the auto-rebaseline carries one way forward and reaches Stuck, `begin-batch`'s terminal-dead refusal names the second-recovery route and `recovery_exhausted`, and the classifier, rebaseline and shell-wait tables gain the rows the landing review called thin.

## Scope

In:

- `internal/websterengine`: the contract-evidence path recover-batch takes over a failed batch's uncheckable entries (`contractevidence.go`, `findingtext.go`, `recoverbatch.go`), the auto-rebaseline's Merriam-base failure text (`autorebaseline.go`, `merriambase.go`, `runlevel.go`), and `begin-batch`'s existing-report remedy (`beginbatch.go`).
- `internal/fabricengine/refscanner.go`: `IsReadOnlyCommand` and its `lyx` and `git` segment rules.
- `contracts/specs/refusal-spec.md`: the rows the changed refusals belong to.
- Tests: rows in the existing tables named under Acceptance criteria; package docs (`doc.go`, the classifier's doc comment) in the same commit.

Out:

- Any change to the audit's class vocabulary, the batch failure path, `accept-audit`'s evidence rules, or the Master stencil's rungs.
- Making `canonicalPath` tolerant of long names or odd characters; the fix keeps non-paths away from it instead.
- A run-time check of which `lyx` verbs run under `--json`; the classifier stays static-shape only.
- Parallel or quarry-side work on other board entries (`cli-surface`, `master-self-fix`, `create-check-unexported`).

## Decisions

### D1. Pathless uncheckable entries never reach the filesystem

Decision: an uncheckable entry recorded for a correctness finding with no path, the `<class>: <detail>` form `failOnCorrectness` writes, is recognised by one predicate beside that writer, and `contractWritesFor`, `splitContractPaths` and `uncheckableReason` skip such an entry: it is no contract file, it goes to `Rest`, and its reason is the existing no-path reason, without a `canonicalPath` call.
The predicate keys on a known audit class followed by `: `, not only the fabric-reference class, so every pathless class shares the rule.
Rationale: #513 is `filepath.EvalSymlinks` run on a shell command, which fails with `ENAMETOOLONG` or any error that is not `ErrNotExist`; only recover-batch passes mixed entries to these functions, and the other callers pass suspect paths, so skipping at the function keeps every caller correct without a second partition at each site.
Bound: the skip applies to entries that carry the class prefix only; a path entry still resolves and still errors on a genuine link failure.

### D2. The classifier tokenises `lyx` and `git` segments shell-style and admits quoted arguments, quoted globs, tilde paths and globs after `--`

Decision: a `lyx` or `git` segment is split into words honouring single and double quotes, adjacent quoted and unquoted parts joining into one word, and every rule below runs on the dequoted word.
The command word and the subcommand words stay unquoted and plain as today.
A word holding `$` or a backslash, quoted or not, or an unquoted brace, rejects the segment.
A dequoted word that starts with `-` is a flag and must match the form exactly as today, so a quoted writing flag is rejected.
An unquoted glob character (`*`, `?`, `[`) is admitted only in a word after `--` in a `git` segment and nowhere in a `lyx` segment; a quoted glob is text and admitted wherever a positional is.
An unquoted leading `~` is admitted in a positional and in `-C`'s directory.
Rationale: `git log --format='%h %s'` is the read #514 halted on, and quoting must be understood rather than banned, because a quote can also hide a flag; a glob expands to names in the working tree, and after `--` git reads every word as a path, so a planted name that spells a flag cannot act there; a tilde expands to an absolute path, which no reader takes as a flag.
Bound: the classifier still sees static shape only; a loop (`for v in ...; do ...; done`), a `$v` expansion, a substitution and a command behind `bash -c` stay non-read-only, which the rows pin.

### D3. A trailing `--json` is a help form

Decision: `readOnlyLyxSegment` admits command words then `--json` as the last word, and command words then `--help` or `-h` then `--json`, under the same command-word rule as the help form today, including bare `lyx --json`.
Rationale: since 57243019b the global `--json` raises `--help` before any pre-run hook or argument check for every command, so nothing runs.
Bound: a verb that declares its own local `--json` shadows the global flag and runs; today that is `lyx shed status` alone, which reads.
A test pins the set of commands declaring a local `--json`, so a new one is a deliberate change reviewed against this rule; the classifier does not know the set.

### D4. Each git reader form admits only flags that cannot write, and a test row per form proves a writing flag is rejected

Decision: the exact-match flag lists and the two prefixes (`--format=`, `--pretty=`) in `refscanner.go` stay the bound; the task reviews every listed flag against git's documentation and removes any that writes, executes or touches config, and the classifier table gains, per reader form, a row rejecting one such flag (`--output` for `log`, `show` and `diff`; `set-url` for `remote`; a rename or upstream flag for `branch`; `--file` for `config`; `--upload-pack` for `ls-remote` already present).
Rationale: the landing review blocked on unbounded reader flags; the current lists look bounded, and the rows turn that into a pinned fact instead of a reading.

### D5. The auto-rebaseline's Merriam-base failure carries one way forward

Decision: `MerriamBase`'s error states the file and the cause and no way forward; each caller words its own: the CLI verbs and the fresh partition keep "transient, re-run the verb" or the run's re-entry step, and `autoRebaseline` wraps it in `ErrAutoRebaseline` with "transient, <re-entry step>" as today, so the message holds exactly one `way forward:` clause.
Rationale: today the two clauses concatenate, and the first names a verb re-run the Webster row's refusal must never name (spec row "Webster row: auto-rebaseline refused"); the Stuck mapping itself already holds through `ErrAutoRebaseline` and the shed adapter, and the existing row "a failed read of Merriam's base" covers it, so the remaining gap is the text.

### D6. `begin-batch`'s existing-report remedy mirrors the recover-batch verdicts for a dead record

Decision: `existingReportRemedy` words the terminal-dead case by the record: a dead fork record names `lyx webster recover-batch NN`; a dead recovery below the cap that committed work of its own names it once more; a dead recovery at the cap names the batch's recoveries as exhausted, the `recovery_exhausted` state, and `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN`, or `1) lyx webster reset --to start; 2) lyx webster run`; a dead recovery below the cap that committed nothing names ending the run stuck naming the batch.
The spec row "report already present" is reworded to the same four cases.
Rationale: today every dead record that earns no retry reads "exhausted", which is false for a dead fork batch that has had no recovery, and names no route for the exhausted case that recover-batch's own refusal names.

### D7. New scenarios land as rows in the existing tables

Decision: every scenario under Acceptance criteria is a row or an assertion in the table that already covers its surface; no new top-level test unless a scenario has no table, in which case the card lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
Rationale: `internal/websterengine` and `internal/fabricengine` sit at their budget rows, and PATTERN-test-economy puts items of one kind in one table.

## Constraints

- PATTERN-refusal-way-forward: D5 and D6 change refusal texts, so their spec rows change in the same commit and a row reaches each.
- PATTERN-no-denied-recovery: the routes D6 names are the ones recover-batch's exhausted refusal already names; no new command.
- PATTERN-test-economy and PATTERN-test-speed: rows, untagged where a fake serves; `begin-batch` and recover-batch rows already run on the git fixture their tables use.
- PATTERN-comment-line-breaks and semantic line breaks in every `.md`.
- PATTERN-documentation-lifecycle: the classifier's forms live in `IsReadOnlyCommand`'s doc comment and the audit section of `internal/websterengine/doc.go`; both change with D2 and D3.
- PATTERN-shuttle-provider-seam, PATTERN-fabric-git: untouched; the classifier never runs a command.
- No PATTERN entry is changed or waived.

## Auto-mode assumptions

- "Merge base" in board item 4 means `MerriamBase`, the Merriam start base, as the finding name `card4-merriambase-failure-escapes-stuck` says; no git merge base runs on the auto-rebaseline path.
- Item 4's Stuck mapping already holds (the adapter maps `ErrAutoRebaseline`; the auto-rebaseline table has a Merriam-base row), so the deliverable is D5's single way forward plus assertions, not a new mapping.
- Item 3's lists are already exact-match; the deliverable is the review plus the rows of D4, with code changes only where the review finds a writing flag.
- The 5a8be0274 design note "a quoted or globbed read stays a correctness halt" is superseded by the board entry, which asks the opposite; D2 keeps its spirit through the dequoted-flag rule and the after-`--` glob bound.
- The thin-row findings named plan cards of 5a8be0274 that no longer exist; item 6 is met by the rows under Acceptance criteria, chosen against the tables as they stand.
- A pathless entry's predicate covers every audit class, not only fabric-reference, since `failOnCorrectness` writes the same form for each.

## Open risks

- A verb gaining a local `--json` later would run under a command the classifier admits; D3's pinning test is the only guard.
- A planted file name spelling a flag could still act where a quoted glob is passed to a reader, since a quoted glob is text to the shell but a pattern to git; git reads such a word as a pathspec, not a flag, so this is theoretical.
- Reworded refusals: any test asserting the old "exhausted" text for a dead fork record or the doubled way forward must change with the code.
- The `cli-surface` run is adding a command-tree form test in `cmd/lyx`; D3's local-`--json` pin may land near it and should sit in `internal/clihelp` if `cmd/lyx` is contested.

## Acceptance criteria

Each scenario is covered by an existing test, a new row or a new assertion.

Pathless findings (D1):

- `RecoverSpawnOrAttach` over a failed batch whose uncheckable entries are a pathless fabric reference with a non-read-only command longer than `NAME_MAX` and a path entry returns the documented `ErrRecoveryNeedsFresh` refusal naming the path entry's reason, with no link-resolution error.
- The same with only pathless entries names the `accept-audit --batch NN` route.
- `splitContractPaths` and `contractWritesFor` over a mix of a contract path and a pathless entry clear or load for the contract path and never resolve the entry.

Classifier (D2, D3, D4), as rows of `TestIsReadOnlyCommand`:

- Admitted: `git log --format='%h %s'`, `git log --format="%h %s" -- '*.go'`, `git log --oneline -- *.go`, `git -C ~/repo status`, `git log ~/x`, `git branch --list 'feat/*'`, `lyx --json`, `lyx fabric add --json`, `lyx fabric --help --json`, `lyx webster record-batch -h --json`.
- Rejected: `git log *.go`, `git log a?`, `git log [a]`, `git log {a,b}`, `git log '--output=x'`, `git log "$X"`, `git log '$X'`, `git log 'a\b'`, `lyx fabric diff '*'`, `lyx fabric $v --help --json`, `for v in a b; do lyx fabric $v --help --json; done`, `lyx fabric add x --apply --json`, `lyx fabric add --json --apply`.
- Per git reader form, one writing or executing flag rejected: `git show --output=x`, `git diff --output=x`, `git remote set-url o u`, `git branch -m a b`, `git config --get --file f k`.

Merriam base (D5):

- `TestRun_AutoRebaseline`'s Merriam-base row asserts exactly one `way forward:` clause, ending in the re-entry step, and no "re-run the verb".
- The CLI path and the fresh partition still name their own way forward (an existing row or one assertion).
- The shed adapter maps the wrapped error to Stuck (existing row).

Begin-batch dead remedy (D6), as rows of the existing-report table:

- A dead fork record names `recover-batch NN` and never "exhausted".
- A dead recovery below the cap that committed names `recover-batch NN` once more (existing row).
- A dead recovery at the cap names "exhausted", `recovery_exhausted`, and the reset-to-batch-start route.
- A dead recovery below the cap that committed nothing names ending the run stuck naming the batch and never "exhausted".

Thin rows (item 6):

- Rebaseline: an accepted auto-rebaseline with only the Card Index of `00-overview.md` changed warns with "accepting no card file"; an accepted one over an amended in-flight card lists it under "amended in-flight cards".
- Shell wait: a transcript-reported and a payload-reported shell ended in one run both land in `EndedShells` with their signal; an awaited shell never lands there.
- Where an existing row already pins one of these, no row is added.

Specs and docs:

- `refusal-spec.md` rows "report already present" and "Merriam start base unreadable" match the new texts; the audit row for fabric references and the `doc.go` audit section describe the classifier's quoted, glob, tilde and `--json` rules.
- `lyx gate test ./internal/websterengine ./internal/fabricengine ./internal/shedadapters ./internal/clihelp ./contracts/specs` green.

## Notes for the plan writer

- The pathless entry form is written at one site in `recordbatch.go` (`failOnCorrectness`); the read side today is `fabricReferencePrefix` in `auditledger.go`.
  One predicate should serve both, and `allPathlessFabricReference` keeps its fabric-only meaning for the accept route.
- Segment boundaries in `IsReadOnlyCommand` are already found with quoted spans blanked (`blankQuoted`), so only the per-segment word split changes; the readers (`cat`, `grep`, ...) keep their no-word-check rule.
- `existingReportRemedy` already receives `RecoveryRetry`'s verdict; it also needs the record's kind and count, both on `BatchState`.
- `merriamBaseError` is the only place the Merriam-base way forward is worded; `RunDeps.reentryStep` and `stepOrRun` give the row and CLI steps.
- The classifier's docs live in three places: the `IsReadOnlyCommand` doc comment, `internal/websterengine/doc.go`'s audit section, and the fabric-reference rows of `refusal-spec.md`.
