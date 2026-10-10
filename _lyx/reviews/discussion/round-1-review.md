---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:62
    summary: D6 names recover-batch for every dead fork record, but a re-begun fork record carries its Recoveries, so at the cap recover-batch refuses exhausted, and D6 also drops the unrendered-amendment exemption recover-batch applies
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:38
    summary: D2 admits a quoted glob wherever a positional is, which admits lyx fabric diff '*', yet Acceptance criteria lists that command as rejected
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:48
    summary: D3's only guard, the local --json pin, has no Acceptance criteria row and no cmd/lyx gate run, and the fallback home internal/clihelp cannot see the real command tree built in package main
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:38
    summary: D2's unquoted-glob bound names no shell dialect and does not reject unquoted parentheses, so under zsh a glob qualifier after -- such as *(e:...:) runs code while the classifier calls it read-only
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:40
    summary: "D2's rationale that after -- git reads every word as a path is false for ls-remote and remote show, where an expanded name becomes a repository or remote name"
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:9
    summary: D5 strips the way forward from MerriamBase's error, but Scope In omits internal/webstercli, whose two callers pass that error straight to the CLI and would then refuse with no way forward
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:84
    summary: Two Auto-mode assumptions restate D1's and D5's own decisions; they belong once, in the Decision
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:93
    summary: Open risks holds a non-risk (a quoted glob never becomes a flag) and a routine test-update note, neither of which the plan writer acts on
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:52
    summary: D4 says only the ls-remote rejection row is already present, but TestIsReadOnlyCommand already holds git log --output=x
---

### [BLOCKING] D6's dead-fork case names a route that refuses at the cap

**Location:** `_lyx/discussion/decision-record.md:62`, with the matching Acceptance row at line 123.

**Issue:** D6 says it mirrors recover-batch's verdicts, but its fork case does not.
`BeginBatch` carries `prior.Recoveries` and `RecoveryStartSHA` into the fresh fork record (`internal/websterengine/beginbatch.go:525-546`: "so a re-begin does not hand a batch a fresh pair of recoveries").
A dead fork record can therefore sit at `maxRecoveries`.
For such a record `RecoverSpawnOrAttach` refuses with `ErrRecoveryExhausted` (`recoverbatch.go:440-442`), so begin-batch's refusal would name `lyx webster recover-batch NN` as the way forward, and that step is itself refused.
That is a refusal whose named way forward does not work, which `PATTERN-refusal-way-forward` forbids, and the spec row would be reworded to match it.
The same gate exempts a record that `rendersNewAmendment`: a recovery at the cap with an unrendered amendment still gets a recovery.
D6's at-cap case names that record exhausted, which is the opposite of recover-batch's verdict.
The four cases are keyed on record kind, but recover-batch decides on count and amendment first and kind second, so a case falls through.
This settles the focus question: the combinations are not all covered.

**Fix:** Key D6 on the order recover-batch uses.
First, count at the cap with no unrendered amendment, for either kind: exhausted, `recovery_exhausted`, and both reset routes.
Then a dead fork record: `recover-batch NN`.
Then a dead recovery with `RecoveryRetry` true: once more.
Otherwise: end the run stuck naming the batch.
Add Acceptance rows for a dead fork record at the cap and for an unrendered amendment at the cap.

### [MEDIUM] D2 admits `lyx fabric diff '*'`, which Acceptance rejects

**Location:** `_lyx/discussion/decision-record.md:38` against line 110.

**Issue:** D2 says "a quoted glob is text and admitted wherever a positional is", and the sentence covers both `lyx` and `git` segments.
`lyx fabric diff` takes one positional not starting with `-` (`readOnlyLyxSegment`), so under D2 the dequoted `*` is admitted.
Acceptance criteria lists `lyx fabric diff '*'` as rejected.
The plan writer reads both and gets two answers.

**Fix:** Decide whether a quoted glob is admitted in a `lyx` segment.
Then state it in D2, or move the Acceptance row to the admitted list.

### [MEDIUM] D3's pinning test has no acceptance row and a fallback home that cannot reach the tree

**Location:** `_lyx/discussion/decision-record.md:48`, `:95`, `:135`, and `support-log.md` question ledger.

**Issue:** D3's only guard is "a test pins the set of commands declaring a local `--json`".
No Acceptance criteria row names it, so `Plan-Write`, which reads only the decision record's criteria for its scenarios, can omit it.
The gate line at 135 runs `./internal/clihelp` but not `./cmd/lyx`.
The real command tree is built by `newRoot` in `cmd/lyx/main.go` (package `main`), which calls `clihelp.InstallJSONHelp`.
`internal/clihelp` cannot import package `main`, so a pin placed there, as Open risks and the question ledger prefer, sees only a synthetic tree and pins nothing about the real set.
The guard also counts only a local `--json`; it does not say whether a persistent `--json` on a group, which shadows the global one the same way, counts too.

**Fix:** Put the pin in `cmd/lyx` against `newRoot()`, covering local and persistent `--json` declarations.
Add an Acceptance row for it and `./cmd/lyx` to the gate line.
Drop the `internal/clihelp` fallback from Open risks and the ledger.

### [MEDIUM] D2's glob bound assumes bash and leaves unquoted parentheses unchecked

**Location:** `_lyx/discussion/decision-record.md:38-41`.

**Issue:** D2 admits an unquoted `*`, `?` or `[` after `--` and bounds it with "a glob expands to names in the working tree".
That holds in bash.
`plainWord` today rejects no `(` or `)`, and D2 adds no rule for them.
Under zsh, `git log -- *(e:'touch x':)` is a glob qualifier that runs the quoted code for each match.
Its quotes dequote cleanly, and it holds no `$`, backslash or brace, so D2 admits it.
The agents' Bash tool runs under the user's shell, which on macOS defaults to zsh.
The Decision states no shell assumption, so the classifier would call this command read-only.

**Fix:** In D2, reject a word holding an unquoted `(` or `)`.
Also state the shell dialect the static-shape bound assumes, so the bound is checkable.

### [LOW] The after-`--` premise does not hold for every git form

**Location:** `_lyx/discussion/decision-record.md:40`.

**Issue:** D2's rationale says "after `--` git reads every word as a path".
`gitForm.admits` returns true at `--` for every form in `gitForms` and for `remote show` and `remote get-url`.
For `ls-remote`, the first word after `--` is the repository, so `git ls-remote -- *` sends a planted name to the transport as a URL or local repository path.
For `remote show`, the words after `--` are remote names.
Neither is shown to write, but the stated bound is false for them, and a later reader relies on it.

**Fix:** Admit unquoted globs after `--` only for the forms whose post-`--` words are pathspecs (`log`, `show`, `diff`, `status`), or narrow the rationale to those forms.

### [LOW] D5 changes an error that `internal/webstercli` passes through unchanged

**Location:** `_lyx/discussion/decision-record.md:9-13` against D5 at line 57.

**Issue:** D5 makes `MerriamBase`'s error carry no way forward and has "the CLI verbs" word their own.
The CLI callers are `executionBatches` (`internal/webstercli/cli.go:298`) and the rebaseline verb (`internal/webstercli/rebaseline.go:98`).
Both return or print the error as-is.
Scope In names only `internal/websterengine` files, so a plan held to Scope leaves the CLI refusals with no way forward.
This is not a call-site enumeration finding: it is the package boundary the scope draws, which decides whether the plan may edit that package at all.

**Fix:** Add `internal/webstercli` to Scope In for D5.
Alternatively, keep a CLI-shaped way forward inside `websterengine`, for example a wrapper the CLI calls.

### [LOW] Auto-mode assumptions restate two Decisions

**Location:** `_lyx/discussion/decision-record.md:84`, `:88`.

**Issue:** "Item 4's Stuck mapping already holds…" restates D5's rationale.
"A pathless entry's predicate covers every audit class…" restates D1's second sentence.
Neither is an assumption made for a missing operator answer.
Both carry no requirement beyond what D1 and D5 already state, so the completeness-before-leanness test is met and they can go.
The other Auto-mode entries (the meaning of "merge base", the superseded design note, the vanished plan cards, item 3's lists) are genuine self-picks and stay.

**Fix:** Delete the two lines from Auto-mode assumptions.

### [NIT] Open risks holds a non-risk and a routine note

**Location:** `_lyx/discussion/decision-record.md:93-94`.

**Issue:** Line 93 calls a planted flag-named file acting through a quoted glob "theoretical", but it cannot happen.
A quoted glob reaches git as one word, and a pattern git matches internally never becomes an argument.
A quoted glob that dequotes to a leading `-` is already rejected by D2's flag rule.
Line 94 says tests asserting old text change with the code, which every refusal rewording implies and which carries no requirement.
Neither is something the plan writer acts on.

**Fix:** Delete both lines.
The D2 reasoning, if worth keeping, goes to `support-log.md`.

### [NIT] D4 miscounts the rows already present

**Location:** `_lyx/discussion/decision-record.md:52`.

**Issue:** D4 lists `--output` for `log` as a row to add and only `--upload-pack` as already present.
`internal/fabricengine/refscanner_test.go:197` already holds `git log --output=x` rejected.

**Fix:** Mark the `log` row as already present.

## Focus items judged sound

- D2, flag-shaped dequoting: a quoted or joined word that dequotes to a leading `-` falls under the exact-match flag rule, and one that dequotes to `--` is the separator to both git and the classifier, so no dequoted word escapes the flag rule.
  The gaps found are F4 and F5, not dequoting.
- D3, flag value `--json`: every word before the trailing `--json` must match `lyxCommandWord`, so no flag can take `--json` as its value.
  A positional that happens to be a lowercase word reaches cobra as an argument, and help still fires before argument checks.
  Bare `lyx --json` and `--help`/`-h` then `--json` raise help; other orderings are not admitted.
  No command declares `DisableFlagParsing` today.
- D1, a real path with a class prefix: violation paths come from tool inputs and are absolute or worktree-relative, and the known classes (`nested-agent`, `fabric-reference`, …) followed by `: ` are not a realistic path start.
  If one did occur, the entry was already uncheckable at record time, and skipping `canonicalPath` changes only its reason text, never whether it blocks.
  Not a defect.
- D5 against Acceptance: the single `way forward:` clause and the Stuck mapping through `ErrAutoRebaseline` (`shedadapters/webster.go:82`) agree, apart from F6.
