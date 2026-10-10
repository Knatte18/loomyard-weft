# Webster audit gaps: pathless findings, read-only classifier, refusals

`recover-batch` no longer fails on a correctness finding that has no path.
The fabric-reference classifier admits more read-only calls, with every admitted git reader's flags bounded against writing.
The Merriam-base failure under auto-rebaseline and `begin-batch`'s existing-report refusal each carry one accurate way forward.

Closes #513
Closes #514
Closes #518

## Behaviour

- **Pathless findings (#513).**
  An uncheckable entry written as `<class>: <detail>` by `failOnCorrectness` is recognised by one predicate beside that writer.
  `contractWritesFor`, `splitContractPaths` and `uncheckableReason` skip such an entry: it goes to `Rest` with the no-path reason and never reaches `canonicalPath`.
  `recover-batch` over a failed batch with a long shell command as its only finding now returns its documented refusal naming `accept-audit --batch NN`, instead of an `ENAMETOOLONG` link error.
  Path entries still resolve and still error on a genuine link failure.
- **Classifier (#514, #518), `IsReadOnlyCommand` in `internal/fabricengine/refscanner.go`.**
  `lyx`, `git` and `sed` segments are split into words honouring single and double quotes, and each rule runs on the dequoted word.
  - A dequoted word starting with `-` is still a flag and must match exactly, so a quoted writing flag is rejected.
  - A word holding `$` or a backslash, or an unquoted brace or parenthesis, rejects the segment.
  - A quoted glob is text.
    An unquoted glob (`*`, `?`, `[`, a leading `^`, `#`) is admitted in `git` only after `--` in `log`, `show`, `diff` and `status`, and is rejected in `lyx`, `ls-remote` and `remote`.
  - A leading unquoted `~` is admitted in a positional and in `-C`'s directory.
  - The unquoted words `2>&1` and `2>/dev/null` are dropped before the redirection scans; every other redirection still rejects.
  - `sed` is read-only in one form only: `-n`, one script word that is `Np` or `N,Mp`, then file words that neither start with `-` nor hold an unquoted glob.
  - `lyx` accepts a trailing `--json`, alone or after `--help`/`-h`, as a help form, because the global `--json` raises help before anything runs.
  - Loops, `$v` expansions, substitutions, zsh glob qualifiers, bash extglob and `bash -c` stay non-read-only.
- **Merriam base under auto-rebaseline.**
  `MerriamBase`'s error now states the file and the cause only; each caller adds its own way forward.
  The auto-rebaseline message holds exactly one `way forward:` clause, the re-entry step, where the two clauses used to concatenate and the first named a verb re-run the Webster row must never name.
  It still maps to Stuck through `ErrAutoRebaseline`.
- **`begin-batch` existing-report refusal.**
  A terminal record that is not done, whose recoveries are at the cap and that holds no unrendered amendment, wraps `ErrRecoveryExhausted` and names the reset routes recover-batch's own refusal names.
  The `begin-batch` verb sets `recovery_exhausted: true` on that case, and Master's begin-batch rung gains one sentence for it.
  A dead recovery that earns `RecoveryRetry` names `recover-batch NN` once more; any other dead recovery ends the run stuck naming the batch and no longer reads "exhausted" below the cap.

## Design decisions to review

- Quoting is understood, not banned, so a quote cannot hide a flag.
  The after-`--` glob bound relies on those four git forms reading every later word as a pathspec, so a planted file name spelling a flag cannot act.
- The classifier assumes bash or zsh with any glob option set and sees static shape only.
  A revision exclusion must be written quoted (`'^main'`) or as a range (`main..feat`).
- The exhausted case at any status carries the flag recover-batch's refusal carries, so `begin-batch` and Merriam never disagree on one state.
- Every new scenario is a row in an existing table.
  The `cmd/lyx` budget row is raised from 72 to 73 for the one new test, and then to 74 to cover both merged sides' new tests.
- `refusal-spec.md` rows "report already present" and "Merriam start base unreadable" are reworded, and `internal/websterengine/doc.go` and the classifier's doc comment describe the new rules.
- No git reader flag list needed a flag removed; the review is pinned by one rejection row per reader form (`git show --output=x`, `git diff --output=x`, `git remote set-url o u`, `git branch -m a b`, `git config --get --file f k`).
- Thin rows added for the auto-rebaseline (card-index-only change, amended in-flight cards) and for ended shells (transcript- and payload-reported shells land in `EndedShells`, an awaited shell does not).

## Check by hand

- A new `--json` of a command's own would shadow the global flag and run under a call the classifier admits.
  The only guard is the `cmd/lyx` test that walks `newRoot()` and pins the commands declaring their own `--json`; the pinned set is `batten status`, `loom status` and `shed status`, where the design assumed `shed status` alone.
  Confirm the extra two are read-only, and watch for a conflict with the `cli-surface` command-tree form test.
- Accepted risks from review that stay open in the design:
  - the glob bound assumes bash or zsh parenthesis forms (unquoted parenthesis rejects the segment) and has no test against a real shell;
  - a dead fork record at the recovery cap is not a case of its own, since only a recovery record is ever terminal dead;
  - Master's rung covers the exhausted refusal only through the one added sentence.
- The `webster-template-master.md` sentence reads correctly against the surrounding rung.
- `internal/loomcli/bootstrap.go` and `internal/shuttleengine/waiting_test.go` also changed, from the merged-in side and the shell-wait rows; the diff there is worth a glance.
