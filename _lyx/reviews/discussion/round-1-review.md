---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md, D2 (Goal item 2, D2 body, Bound and Acceptance criteria)
    summary: D2 rests on the premise that an unquoted parenthesis is the only way a reader segment runs code, but reader segments also admit unquoted and double-quoted dollar expansions, which run code under bash and zsh, and D2 states no bound for them
    fix: In decision-record.md D2, add a decision on dollar expansions in reader segments. Recommended - a reader segment is also not read-only when, in its view with only single-quoted spans blanked (the view the substitution scan already uses), a dollar sign is followed by an opening brace, an opening square bracket or a name character, so parameter and arithmetic expansions are rejected while a regex anchor such as grep 'a$' f or grep "^a$" f stays read-only. Then in the same D2 paragraph name the guards that already cover reader segments at command level (the command and process substitution markers, every output redirection, backgrounding) and state that input redirection with < stays admitted because it only reads. Extend Goal item 2 and the Scope In bullet on IsReadOnlyCommand to name the dollar rule, and change the Scope Out bullet to say no change beyond the parenthesis and dollar rules. Add Acceptance criteria rows - IsReadOnlyCommand false for cat ${x@P}, head -n ${a[y]} f and cat $x; true for grep 'a$' f and grep "^a$" f. State that the IsReadOnlyCommand doc comment's sentence that an expansion stays non-read-only becomes true for readers by this rule. If the writer instead keeps dollar expansions free, D2 must state that bound explicitly (no admitted segment assigns a variable, so an expansion can run code only from a value set before the command, such as an environment variable) and the doc comment sentence on expansions must be corrected to except readers.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/discussion/decision-record.md, D1 Bound
    summary: D1's list of figures the cutoff gates omits the run's plan, which -calibrate picks as the newest plan commit before the run's first Webster fork, so with -no-cutoff an earlier run's fork can select an earlier run's plan or none
    fix: In decision-record.md D1 Bound, add the run's plan to the list of gated figures - it is read from the newest plan commit before the run's first Webster fork, so with the flag an earlier run's fork can move that point back and select an earlier run's plan, or none. Extend the sentence on the doc comment so the flag's description tells a -calibrate user that the plan, and with it the peak table, the start table's base and the identity card counts, may come from an earlier run, or the run may be listed with no plan commit before the first fork.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md, D1 body and Bound
    summary: D1 does not say that with -no-cutoff a main-session transcript with no timestamped user or assistant line is counted, though today it is left out whenever a bound exists
    fix: In decision-record.md D1 Bound, add one clause - with the flag, a transcript with no timestamped user or assistant line is counted too, since the zero bound skips the start check. The flag's doc-comment description says the same.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md, Question ledger
    summary: Q7 is listed between Q4 and Q5, out of numeric order
    fix: In support-log.md's Question ledger, move the Q7 bullet below the Q6 bullet.
---

### [BLOCKING] Dollar expansions in reader segments run code and D2 bounds none of them

**Location:** `_lyx/discussion/decision-record.md`, D2, with Goal item 2, Scope and Acceptance criteria.

**Issue:** D2 keeps readers out of `splitShellWords` precisely because that split rejects `$`, so `readOnlySegment` (`internal/fabricengine/refscanner.go:298`) returns true for any reader segment holding an unquoted or double-quoted `$` expansion.
Only the markers `$(`, a backtick, `<(` and `>(` are rejected command-wide (`refscanner.go:198`, `:222-227`).
Bash runs code from a parameter expansion without any of those markers; I checked both forms in this environment:
`x='$(echo RAN >&2)'; echo "[${x@P}]"` prints `RAN`, and `y='a[$(echo RAN2 >&2)0]'; a=(z); head -n ${a[y]} /dev/null` prints `RAN2`, the arithmetic subscript evaluating the variable's value.
Under zsh with GLOB_SUBST, which the classifier's "any glob option set" assumption includes, `cat $x` expands a qualifier glob held in `x`.
No admitted segment can assign a variable, so the value must exist before the command, but that is a bound D2 never states; D2 instead says the doc paragraph will explain "why reader globs and flags need no other bound", and the Goal says the change closes "the one reader-glob path that can run code".
The existing `IsReadOnlyCommand` comment (`refscanner.go:216`) already claims "an expansion ... stay[s] non-read-only", which is false for readers today, and no test row pins a reader with `$` either way.
This is the attack-surface question from the rubric: D2 consciously retains an edge through the guard and states neither what it lets through nor what bounds it, and the doc paragraph it mandates would record a false safety claim.
D2 also never says that the command-wide substitution, output-redirection and backgrounding checks are what bound backticks, process substitution and redirections in reader segments; that sentence belongs in the same fix, and with it those three vectors are bounded today (the rule I checked against at `refscanner.go:222-237`).

### [MEDIUM] D1's gated-figure list misses the run's plan under -calibrate

**Location:** `_lyx/discussion/decision-record.md`, D1 Bound.

**Issue:** `readRunPlan` (`tools/tokencount/calibrate.go:465-480`) takes the newest `loom: plan artifacts for <slug>` commit made before `earliestFork(run.Forks)`.
With `-no-cutoff`, an earlier run's Webster forks join `run.Forks`, so the first fork can move before the current run's plan commit and the calibration reads the earlier run's plan, or reports "no plan commit before the first fork".
That plan feeds the peak table, the start table's base through `00-overview.md`, and the identity-profile card counts in the batch-cost section.
D1 lists "the begin-batch base" and says only "the base and Merriam rows may come from an earlier run", so an operator reading the flag's description is not told that whole peak-table rows can be judged against the wrong plan, for example as "names a card the plan lacks".
The focus asked whether every gated figure is named; this one is not.

### [LOW] Untimestamped transcripts start counting under -no-cutoff

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** `CountRun` (`tools/tokencount/count.go:723-733`) runs the start check only for a non-zero bound, and leaves out a transcript with no timestamped user or assistant line as part of that check.
The `main.go` doc comment says such a transcript "is left out when a bound exists".
With D1's zero bound it is counted, a change D1's Bound does not mention, though it is small: such a transcript carries no timestamped user or assistant line and so little or no usage.

### [NIT] Question ledger out of order

**Location:** `_lyx/discussion/support-log.md`, Question ledger.

**Issue:** Q7 sits between Q4 and Q5.

## Verdicts on the focus directive's questions

- D2 attack surface: backticks, `$(`, process substitution and output redirections are already rejected command-wide before any segment is judged, and input redirection only reads, so those are bounded by existing code, but D2 does not say so; `$` parameter and arithmetic expansion is open and unbounded (F1).
- D2 premise on reader flags: holds against the GNU flag sets as far as I read them; the Acceptance criteria pin the everyday reads it relies on (`cat *`, the `--include=*.go` grep, the quoted `jq` map) and the doc paragraph carries the admission rule. The premise is a documentation claim, not a test, which the Open risk already names.
- D1 bound: matches what the flag changes except the plan selection (F2) and untimestamped transcripts (F3); the `-calibrate` effect is placed in the flag's doc comment, where the operator reads it.
- D1 and D3 premises about the tree: the D1 skew trigger sits in Rationale and the plan does not depend on it. The D3 claim that every root-file `loomyard-weft` is word-bounded holds for `CLAUDE.md:3` and `README.md:13,148,151,153,161`, the host-phrase scope excludes root files (`internal/lyxcwd/enforcement_test.go:635`), and the criterion that `fabricVocabularyFailures` reports no failure on the current tree makes the plan re-verify it. Not a defect.
- Acceptance criteria: observable and consistent with D1 to D3; no existing true-expected reader row holds an unquoted parenthesis, so the new rule breaks none. The `IsReadOnlyCommand` rows need F1's additions.
- Relocation: nothing in Decisions, Auto-mode assumptions or Open risks is deliberation the plan writer could lose, and no requirement lives only in `support-log.md`. No move raised.
