# Decision record — webster-audit-gaps

## Goal

Close the six webster audit and recovery gaps the board entry lists: a pathless audit finding no longer breaks `recover-batch` (#513), the read-only classifier admits help-only `lyx` calls under `--json` and quoted or globbed reader arguments without admitting expansions or loops (#514), and a compound of reads whose segments carry a stderr redirection or a `sed -n` line print (#518), every admitted git reader's flags are bounded against writing, a failed Merriam-base read under the auto-rebaseline carries one way forward and reaches Stuck, `begin-batch`'s terminal-dead refusal names the second-recovery route and `recovery_exhausted`, and the classifier, rebaseline and shell-wait tables gain the rows the landing review called thin.

## Scope

In:

- `internal/websterengine`: the contract-evidence path recover-batch takes over a failed batch's uncheckable entries (`contractevidence.go`, `findingtext.go`, `recoverbatch.go`), the auto-rebaseline's Merriam-base failure text (`autorebaseline.go`, `merriambase.go`, `runlevel.go`), and `begin-batch`'s existing-report remedy (`beginbatch.go`).
- `internal/webstercli`: the verbs that pass `MerriamBase`'s error through (the fresh partition and the rebaseline verb), which word their own way forward under D5, and the begin-batch verb's `recovery_exhausted` envelope flag (D6).
- `contracts/stencils/webster/webster-template-master.md`: one sentence in the begin-batch existing-report rung (D6).
- `internal/fabricengine/refscanner.go`: `IsReadOnlyCommand`, its `lyx` and `git` segment rules, its stderr-redirection words and its `sed -n` form (D2).
- `cmd/lyx`: the pin of the commands declaring their own `--json` (D3).
- `contracts/specs/refusal-spec.md`: the rows the changed refusals belong to.
- Tests: rows in the existing tables named under Acceptance criteria; package docs (`doc.go`, the classifier's doc comment) in the same commit.

Out:

- Any change to the audit's class vocabulary, the batch failure path, `accept-audit`'s evidence rules, or the Master stencil's rungs other than the begin-batch sentence D6 adds.
- Making `canonicalPath` tolerant of long names or odd characters; the fix keeps non-paths away from it instead.
- A run-time check of which `lyx` verbs run under `--json`; the classifier stays static-shape only.
- Parallel or quarry-side work on other board entries (`cli-surface`, `master-self-fix`, `create-check-unexported`).

## Decisions

### D1. Pathless uncheckable entries never reach the filesystem

Decision: an uncheckable entry recorded for a correctness finding with no path, the `<class>: <detail>` form `failOnCorrectness` writes, is recognised by one predicate beside that writer, and `contractWritesFor`, `splitContractPaths` and `uncheckableReason` skip such an entry: it is no contract file, it goes to `Rest`, and its reason is the existing no-path reason, without a `canonicalPath` call.
The predicate keys on a known audit class followed by `: `, not only the fabric-reference class, so every pathless class shares the rule.
Rationale: #513 is `filepath.EvalSymlinks` run on a shell command, which fails with `ENAMETOOLONG` or any error that is not `ErrNotExist`; only recover-batch passes mixed entries to these functions, and the other callers pass suspect paths, so skipping at the function keeps every caller correct without a second partition at each site.
Bound: the skip applies to entries that carry the class prefix only; a path entry still resolves and still errors on a genuine link failure.

### D2. The classifier tokenises `lyx`, `git` and `sed` segments shell-style and admits quoted arguments, quoted globs, tilde paths, globs after `--`, stderr redirections and `sed -n` line prints

Decision: a `lyx`, `git` or `sed` segment is split into words honouring single and double quotes, adjacent quoted and unquoted parts joining into one word, and every rule below runs on the dequoted word.
The command word and the subcommand words stay unquoted and plain as today.
A word holding `$` or a backslash, quoted or not, or an unquoted brace or parenthesis, rejects the segment.
A dequoted word that starts with `-` is a flag and must match the form exactly as today, so a quoted writing flag is rejected.
An unquoted `^` starting a word, and an unquoted `#` anywhere in a word, count as glob characters, since zsh's `EXTENDED_GLOB` reads a leading `^` as a negated glob over every name in the directory and `x#` as zero or more `x`, so `x#--output=f` matches a name spelling a flag.
In a `git` segment, an unquoted glob character (`*`, `?`, `[`, leading `^`, `#`) is admitted only in a word after `--` in `log`, `show`, `diff` and `status`, the forms whose words after `--` are pathspecs, and a quoted glob is text and admitted wherever a positional is.
In a `lyx` segment, a glob character rejects the word, quoted or not.
An unquoted leading `~` is admitted in a positional and in `-C`'s directory.
An unquoted word spelled exactly `2>&1` or `2>/dev/null`, in any segment after its command word, is dropped before the redirection, backgrounding and segment rules run; every other redirection still rejects the command, the same two spelled with a space or another descriptor included.
`sed` joins the readers in one form only: `-n`, then one script word that is a line number or a range of two and `p` (`12p`, `285,300p`), then file words, none starting with `-` and each under the word rules above, an unquoted glob rejected.
So a `lyx` help call piped into a reader, with `2>&1`, inside a compound of read-only segments is read-only, as #518's `sed -n 285,300p f.md | cut -c1-200; grep -n "x" -r contracts 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30` is.
Rationale: `git log --format='%h %s'` is the read #514 halted on, and #518's compound is the read batch 01 of run quick-recipe halted on; and quoting must be understood rather than banned, because a quote can also hide a flag; a glob expands to names in the working tree, and after `--` those four forms read every word as a pathspec, so a planted name that spells a flag cannot act there; a tilde expands to an absolute path, which no reader takes as a flag;
`2>&1` duplicates a descriptor and `2>/dev/null` discards, so neither writes a file;
GNU `sed` reads a flag after its file operands, so a file word starting with `-`, or a glob that a planted `-i` name could fill, would turn the print into an in-place edit, and a script other than a line print can write (`w`) or execute (`e`).
Bound: the classifier assumes bash or zsh, with any glob option set, and sees static shape only; a revision exclusion is written quoted (`'^main'`) or as a range (`main..feat`); a loop (`for v in ...; do ...; done`), with or without a redirection or a pipe into a reader, a `$v` expansion, a substitution, a zsh glob qualifier or bash extglob (both need an unquoted parenthesis) and a command behind `bash -c` stay non-read-only, which the rows pin.
After `--`, `ls-remote` and `remote` read repository or remote names, so an unquoted glob there is rejected.

### D3. A trailing `--json` is a help form

Decision: `readOnlyLyxSegment` admits command words then `--json` as the last word, and command words then `--help` or `-h` then `--json`, under the same command-word rule as the help form today, including bare `lyx --json`.
Rationale: since 57243019b the global `--json` raises `--help` before any pre-run hook or argument check for every command, so nothing runs.
Bound: a command that declares its own `--json`, local or persistent on a group, shadows the global flag and runs; today that is `lyx shed status` alone, which reads.
A test in `cmd/lyx` walks the real tree `newRoot` builds and pins the set of commands declaring a `--json` of their own, local or persistent, so a new one is a deliberate change reviewed against this rule; the classifier does not know the set.

### D4. Each git reader form admits only flags that cannot write, and a test row per form proves a writing flag is rejected

Decision: the exact-match flag lists and the two prefixes (`--format=`, `--pretty=`) in `refscanner.go` stay the bound; the task reviews every listed flag against git's documentation and removes any that writes, executes or touches config, and the classifier table gains, per reader form, a row rejecting one such flag (`--output` for `show` and `diff`; `set-url` for `remote`; a rename or upstream flag for `branch`; `--file` for `config`; `--output` for `log` and `--upload-pack` for `ls-remote` already present).
Rationale: the landing review blocked on unbounded reader flags; the current lists look bounded, and the rows turn that into a pinned fact instead of a reading.

### D5. The auto-rebaseline's Merriam-base failure carries one way forward

Decision: `MerriamBase`'s error states the file and the cause and no way forward; each caller words its own: the CLI verbs and the fresh partition keep "transient, re-run the verb" or the run's re-entry step, and `autoRebaseline` wraps it in `ErrAutoRebaseline` with "transient, <re-entry step>" as today, so the message holds exactly one `way forward:` clause.
Rationale: today the two clauses concatenate, and the first names a verb re-run the Webster row's refusal must never name (spec row "Webster row: auto-rebaseline refused"); the Stuck mapping itself already holds through `ErrAutoRebaseline` and the shed adapter, and the existing row "a failed read of Merriam's base" covers it, so the remaining gap is the text.

### D6. `begin-batch`'s existing-report remedy never names a route recover-batch refuses, and follows `RecoveryRetry` for a dead recovery

Decision: `existingReportRemedy` words every terminal record that is not done in recover-batch's own order, the first matching case winning:
a record of any kind and any terminal status but done, whose recoveries are at the cap and that holds no unrendered amendment, wraps `ErrRecoveryExhausted` and names the batch's recoveries as exhausted, ending the run stuck naming the batch, and, for the operator, `1) lyx webster reset --to batch-start --batch NN; 2) lyx webster recover-batch NN`, or `1) lyx webster reset --to start; 2) lyx webster run`;
a dead recovery that earns `RecoveryRetry` names it once more;
any other dead recovery names ending the run stuck naming the batch;
a stuck or failed record names `lyx webster recover-batch NN` as today.
The begin-batch verb sets `recovery_exhausted: true` on a refusal wrapping `ErrRecoveryExhausted`, as recover-batch's verb does, and Master's begin-batch existing-report rung gains one sentence: a refusal with `{"recovery_exhausted": true}` ends the run stuck naming the batch, as the recover-batch rung does, and its reset steps are the operator's.
The spec row "report already present" is reworded to the same four cases.
Only a recovery record is ever terminal dead: record-batch records a fork as done, stuck or failed, and only recover-batch's wait classifies a strand dead, so no case keys on a dead fork record.
Rationale: today every dead recovery that earns no retry reads "exhausted", which is false below the cap, and names no route for the exhausted case that recover-batch's own refusal names;
the stuck and failed branch names recover-batch without the count, so at the cap it names a route that refuses.
A re-begun fork record carries the batch's recovery count, and recover-batch refuses at the cap unless an amendment is unrendered, so a case keyed on kind or status before count would name a route that refuses.
A dead recovery follows `RecoveryRetry` alone, as Merriam's ladder does (`recovery_retry: false` stops stuck); an unrendered amendment changes no case but the exhausted one, which it skips because recover-batch admits it.
The exhausted case at any status carries the same flag recover-batch's refusal does, and Merriam stops stuck on it without running a reset its no-other-reset rule forbids, so begin-batch and Merriam never disagree on one state.
Bound: the flag is set only on the exhausted case, and it lets Merriam skip nothing: it stops the run where recover-batch would refuse anyway.

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
- Item 3's lists are already exact-match; the deliverable is the review plus the rows of D4, with code changes only where the review finds a writing flag.
- The 5a8be0274 design note "a quoted or globbed read stays a correctness halt" is superseded by the board entry, which asks the opposite; D2 keeps its spirit through the dequoted-flag rule and the after-`--` glob bound.
- The thin-row findings named plan cards of 5a8be0274 that no longer exist; item 6 is met by the rows under Acceptance criteria, chosen against the tables as they stand.

## Open risks

- A verb gaining its own `--json` later would run under a command the classifier admits; D3's pinning test is the only guard.
- The `cli-surface` run is adding a command-tree form test in `cmd/lyx`; D3's pin may land near it, and a conflict there is resolved at landing, since only `cmd/lyx` sees the real tree.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d2-glob-bound-assumes-bash-parens-unchecked (class design, severity BLOCKING, open in rounds 1, 2, 3)
  - d6-dead-fork-recover-batch-refuses-at-cap (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - d6-exhausted-remedy-uncovered-by-merriam-rung (class decision, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

Each scenario is covered by an existing test, a new row or a new assertion.

Pathless findings (D1):

- `RecoverSpawnOrAttach` over a failed batch whose uncheckable entries are a pathless fabric reference with a non-read-only command longer than `NAME_MAX` and a path entry returns the documented `ErrRecoveryNeedsFresh` refusal naming the path entry's reason, with no link-resolution error.
- The same with only pathless entries names the `accept-audit --batch NN` route.
- `splitContractPaths` and `contractWritesFor` over a mix of a contract path and a pathless entry clear or load for the contract path and never resolve the entry.

Classifier (D2, D3, D4), as rows of `TestIsReadOnlyCommand`:

- Admitted: `git log --format='%h %s'`, `git log --format="%h %s" -- '*.go'`, `git log --oneline -- *.go`, `git -C ~/repo status`, `git log ~/x`, `git branch --list 'feat/*'`, `git log '^main' feat`, `lyx --json`, `lyx fabric add --json`, `lyx fabric --help --json`, `lyx webster record-batch -h --json`, `lyx fabric --help 2>&1 | head -30`, `sed -n 285,300p f.md | cut -c1-200; grep -n "x" -r contracts 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30` (#518), `git log --oneline -- x#`.
- Rejected: `git log x#--output=f`, `lyx fabric --help 2>x`, `lyx fabric --help 2> /dev/null`, `lyx fabric --help 1>&2`, `sed -i 1p f`, `sed -n 1p f -i`, `sed -n 1p *`, `sed -n 1w x f`, `for v in a b; do lyx fabric $v --help 2>&1 | head -5; done`, `git log ^main`, `git log *.go`, `git log a?`, `git log [a]`, `git log {a,b}`, `git log '--output=x'`, `git log "$X"`, `git log '$X'`, `git log 'a\b'`, `git log -- *(.)`, `git log -- @(a|b)`, `git ls-remote -- *`, `git remote show -n -- *`, `lyx fabric diff '*'`, `lyx fabric $v --help --json`, `for v in a b; do lyx fabric $v --help --json; done`, `lyx fabric add x --apply --json`, `lyx fabric add --json --apply`.
- Per git reader form, one writing or executing flag rejected: `git show --output=x`, `git diff --output=x`, `git remote set-url o u`, `git branch -m a b`, `git config --get --file f k`.

Own `--json` pin (D3), in `cmd/lyx`:

- A walk of `newRoot()` lists every command declaring a `--json` of its own, local or persistent, and the list equals the pinned set, today `shed status`.

Merriam base (D5):

- `TestRun_AutoRebaseline`'s Merriam-base row asserts exactly one `way forward:` clause, ending in the re-entry step, and no "re-run the verb".
- The CLI path and the fresh partition still name their own way forward (an existing row or one assertion).
- The shed adapter maps the wrapped error to Stuck (existing row).

Begin-batch dead remedy (D6), as rows of the existing-report table:

- A dead recovery below the cap that committed names `recover-batch NN` once more (existing row).
- A dead recovery at the cap with an unrendered amendment names ending the run stuck naming the batch and never "exhausted".
- A stuck record at the cap with no unrendered amendment wraps `ErrRecoveryExhausted` and names "exhausted", ending the run stuck, and the reset-to-batch-start route.
- A failed fork record at the cap with an unrendered amendment names `recover-batch NN` and never "exhausted".
- A dead recovery at the cap with no unrendered amendment wraps `ErrRecoveryExhausted` and names "exhausted", ending the run stuck, and the reset-to-batch-start route.
- A dead recovery below the cap that committed nothing names ending the run stuck naming the batch and never "exhausted".
- The begin-batch verb's envelope carries `recovery_exhausted: true` on the exhausted case and not on the others (one assertion in its existing refusal test).

Thin rows (item 6):

- Rebaseline: an accepted auto-rebaseline with only the Card Index of `00-overview.md` changed warns with "accepting no card file"; an accepted one over an amended in-flight card lists it under "amended in-flight cards".
- Shell wait: a transcript-reported and a payload-reported shell ended in one run both land in `EndedShells` with their signal; an awaited shell never lands there.
- Where an existing row already pins one of these, no row is added.

Specs and docs:

- `refusal-spec.md` rows "report already present" and "Merriam start base unreadable" match the new texts; the audit row for fabric references and the `doc.go` audit section describe the classifier's quoted, glob, tilde, `--json`, stderr-redirection and `sed -n` rules;
the `doc.go` recovery section says begin-batch's exhausted refusal carries `recovery_exhausted`, and Master's begin-batch rung holds D6's sentence.
- `lyx gate test ./internal/websterengine ./internal/fabricengine ./internal/shedadapters ./internal/webstercli ./internal/clihelp ./cmd/lyx ./contracts/specs` green.

## Notes for the plan writer

- The pathless entry form is written at one site in `recordbatch.go` (`failOnCorrectness`); the read side today is `fabricReferencePrefix` in `auditledger.go`.
  One predicate should serve both, and `allPathlessFabricReference` keeps its fabric-only meaning for the accept route.
- Segment boundaries in `IsReadOnlyCommand` are already found with quoted spans blanked (`blankQuoted`), so only the per-segment word split changes; the readers (`cat`, `grep`, ...) keep their no-word-check rule, and `sed` is not one of them.
  The `>` and `&` scans run over the whole command before segments are split, so D2's two redirection words are removed before those scans, not inside the segment rules.
- `existingReportRemedy` already receives `RecoveryRetry`'s verdict; it also needs the record's count and unrendered amendments (`rendersNewAmendment`), both on `BatchState`.
  The exhausted case must reach the caller as an error wrapping `ErrRecoveryExhausted`, so the begin-batch refusal changes from one `fmt.Errorf` with a remedy string to a branch for that case; the begin-batch verb already maps `ErrPlanDrifted` to a flag the same way.
- `merriamBaseError` is the only place the Merriam-base way forward is worded; `RunDeps.reentryStep` and `stepOrRun` give the row and CLI steps.
- The classifier's docs live in three places: the `IsReadOnlyCommand` doc comment, `internal/websterengine/doc.go`'s audit section, and the fabric-reference rows of `refusal-spec.md`.
