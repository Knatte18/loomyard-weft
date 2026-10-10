---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: D6 treats an unrendered amendment as a retry only at the cap, so a dead recovery below the cap with an amendment and no commit is told to end stuck, and the at-cap case disagrees with Merriam's recovery_retry ladder
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:44
    summary: D2's bash-or-zsh bound misses zsh EXTENDED_GLOB, under which an unquoted leading ^ (admitted today and under D2, and a common git log idiom) expands to every name in the directory before --, flag-spelling names included
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:64
    summary: D6 fixes only the dead branch; the default branch still names recover-batch for a terminal stuck record at the cap with no unrendered amendment, which recover-batch refuses as exhausted
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:130
    summary: the D6 acceptance rows omit a dead fork record at the cap with an unrendered amendment, the one fork case where count and amendment interact
---

### [MEDIUM] D6 counts an amendment as a retry at the cap but not below it

**Location:** `_lyx/discussion/decision-record.md:64` (D6, cases three and four)

**Issue:**
D6 says it mirrors recover-batch's verdicts.
`RecoverSpawnOrAttach` (`internal/websterengine/recoverbatch.go:440`) refuses only on `Recoveries >= maxRecoveries && !rendersNewAmendment(prior)`.
Below the cap it admits every dead record, and `recoverSpawn` does not count a spawn that renders a new amendment (`recoverbatch.go:345`).
D6's case three names recover-batch for a dead recovery "that earns `RecoveryRetry`, or that is at the cap with an unrendered amendment".
Case four sends every other dead recovery to "end the run stuck".
So a dead recovery below the cap that committed nothing and holds an unrendered amendment ends stuck,
yet the same record one count higher, at the cap, gets recover-batch.
Recover-batch admits both, and neither spawn counts.
The amendment admits a retry in one case and not the other, with nothing in D6's rationale to explain the difference.

The at-cap case also disagrees with Merriam.
`RecoveryRetry` returns false at the cap (`recoverbatch.go:383`).
The package doc says Merriam "runs the second recovery on recovery_retry: true and stops stuck on false or on recovery_exhausted" (`internal/websterengine/doc.go:383`).
Scope Out leaves the Master stencil's rungs untouched.
For one state, begin-batch's remedy then names recover-batch while Merriam's ladder stops stuck.

**Fix:**
Pick one rule for an unrendered amendment on a dead recovery and state it in D6.
Either the amendment earns the uncounted re-run at any count, so case three reads "earns `RecoveryRetry`, or holds an unrendered amendment", plus an acceptance row for the below-cap amendment case.
Or the remedy follows `RecoveryRetry` alone and the at-cap amendment case ends stuck as Merriam does.
Under the first choice, also say how Merriam's `recovery_retry: false` stop squares with it: through `RecoveryRetry` itself, or an Auto-mode assumption that the ladder's disagreement is accepted.

### [MEDIUM] D2's shell bound misses zsh EXTENDED_GLOB's leading `^`

**Location:** `_lyx/discussion/decision-record.md:44` (D2 Bound)

**Issue:**
D2's bound is "the classifier assumes bash or zsh", and its threat model is "a planted name that spells a flag".
D2 closes the glob characters to `*`, `?` and `[`, plus unquoted parentheses and braces.
Under zsh with `EXTENDED_GLOB` set, an unquoted word starting with `^` is a negated glob matching every name in the directory but the rest.
The agents' Bash tool runs the operator's own shell with their profile, so that option comes along when it is set.
`plainWord` (`internal/fabricengine/refscanner.go:259`) admits `^` today, and D2 does not reject it.
`git log ^main feat` is a normal revision-exclusion read.
Before `--`, that zsh expansion hands `git log` every name in the cwd, so a planted file named `--output=x` arrives as a writing flag.
`historyForm.admits` then sees only the original word `^main`, which is not a flag, so it is admitted.
The other EXTENDED_GLOB operators (`x~y`, `x#`) match only names that share the word's literal prefix, so they cannot spell a flag.
Only a `^` at the start of a word, or after a `/`, can.

**Fix:**
In a `git` or `lyx` segment, reject an unquoted `^` at the start of a word before `--`.
A quoted `'^main'` stays admitted, since D2 already treats quoted text as literal.
Add rejected and admitted rows to `TestIsReadOnlyCommand`, for example rejected `git log ^main` and admitted `git log '^main' feat`.
Alternatively, narrow the bound to "bash, or zsh without EXTENDED_GLOB" and record the residual under Open risks.

### [LOW] D6 leaves the same wrong route in the default branch

**Location:** `_lyx/discussion/decision-record.md:64` (D6); code at `internal/websterengine/beginbatch.go:341`

**Issue:**
D6's rationale is that "a case keyed on kind before count would name a route that refuses".
`existingReportRemedy`'s default branch, which covers terminal stuck and failed records, names `lyx webster recover-batch NN` without looking at the count.
A stuck record keeps its report on disk, so begin-batch's existing-report guard reaches it.
A batch re-begun after two recoveries carries `Recoveries` at the cap (`beginbatch.go:545`), and recover-batch refuses it with `ErrRecoveryExhausted`.
A failed record mostly avoids this, because `failBatch` archives the report, but a late report reaches the same branch.
The operator still arrives through the exhausted refusal's own route, so this costs a hop rather than leaving no way forward.

**Fix:**
Either extend D6's at-cap-without-amendment case to every terminal status recover-batch would refuse, and reword the spec row "report already present" to match.
Or record under Scope Out that only the dead branch changes, and why.

### [LOW] D6 acceptance rows skip the fork-at-cap-with-amendment case

**Location:** `_lyx/discussion/decision-record.md:130`

**Issue:**
Rebaseline can add an unrendered amendment to an in-flight fork record (`internal/websterengine/rebaseline.go:378`), and a re-begun fork carries the count (`beginbatch.go:545`).
A dead fork at the cap with an unrendered amendment therefore has to skip D6's first case and reach the second.
That ordering is the one an implementation keyed on count alone gets wrong.
The rows cover a fork below the cap and a fork at the cap without an amendment, but not this case.

**Fix:**
Add the row: a dead fork record at the cap with an unrendered amendment names `recover-batch NN` and never "exhausted".

## Verdicts on the focus items with no finding

- D6, the remaining cases: reached and judged against `RecoverSpawnOrAttach`'s gate order, which is: cap and amendment, then the failed-only evidence checks, then the delete-reference check, then `refuseRecoveringDoneReport`.
  The evidence checks and the done-report guard both pass a terminal dead record.
  A dead fork below the cap, and a dead record of either kind at the cap with an amendment, reach a spawn.
  At the cap without an amendment, the reset-to-batch-start route clears the count and the start, then recover-batch admits.
  `begin-batch` carries `Recoveries` but not `AmendedCards` into a re-begun fork, so after a re-begin only a rebaseline during the new fork can make that fork's amendment unrendered.
  D6's rationale states the carry correctly.
- D3: not a defect.
  The global `--json` raises `--help` through `jsonHelpValue.Set`, and cobra checks help before every pre-run hook (`internal/clihelp/jsonhelp.go`).
  No command sets `DisableFlagParsing`, and `newRoot`'s `PersistentPreRunE` never runs on the help path.
  The only command declaring its own `json` flag is `shed status` (`internal/shedverbs/status.go:230`).
  The pin sits in `cmd/lyx` against `newRoot()`, the only package that sees the real tree.
  The acceptance list has a row for it, and the gate line includes `./cmd/lyx`.
  `cmd/lyx` has 71 top-level tests against a budget row of 72.
- D2: the quoted-glob rule for `lyx` segments agrees with the row `lyx fabric diff '*'` rejected.
  Besides the four pathspec forms, `gitForm.admits` stops at `--` for `ls-remote`, `rev-parse` and both `remote` forms.
  D2 names `ls-remote` and `remote` as rejected.
  `rev-parse` is left out of the closed list of four, so it is rejected too; a row would make that visible, but its absence is no defect.
  Words with unquoted parentheses, which zsh glob qualifiers, bash extglob and subshells all need, are rejected.
- D5: not a defect.
  `MerriamBase`'s callers are `newPartition` (`internal/websterengine/runlevel.go:461`), `autoRebaseline` (`autorebaseline.go:49`), `executionBatches` (`internal/webstercli/cli.go:298`) and the rebaseline verb (`internal/webstercli/rebaseline.go:98`).
  All four are in packages Scope In names.
  No other package's text or test pins the Merriam-base way forward: the only text pins are `merriambase_test.go:104` and the spec rows at `contracts/specs/refusal-spec.md:157-158`.
