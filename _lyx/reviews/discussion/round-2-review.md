---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:58
    summary: ANSI-C quoting defeats the command-wide substitution scan D2 names as its bound, so a reader segment still runs code under the new rules
    fix: In D2, make the dollar rule also reject a `$` followed by a single quote in a reader segment's single-quote-blanked view (the ANSI-C quote `$'…'`, which the quote scanner mis-parses), and say in the paragraph at line 58 that the command-wide substitution scan relies on this, since `blankQuotedSpans` treats `\'` inside `$'…'` as a closing quote. Add `cat $'\'' $(touch x) f` to the false list of the IsReadOnlyCommand Acceptance criteria row, and keep `grep 'a$' f` and `grep "^a$" f` true. Reword Goal 2's 'closes the two reader paths that can run code without a substitution marker' so it does not claim those two are the only paths.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: The dollar rule lists the followers it rejects, so zsh's $~x (glob substitution on, the same qualifier vector as cat $x), $=x, $^x, $+x and positional parameters $0-$9 stay read-only
    fix: In D2 line 54, invert the rule to deny by default - a reader segment is not read-only when, in its single-quote-blanked view, a `$` is followed by anything other than whitespace, a double quote or the end of the segment - and restate the regex-anchor sentence against that rule. Name zsh's `$~x` in the D2 premise paragraph beside `cat $x`, and add `cat $~x` and `cat $1` to the false list of the IsReadOnlyCommand Acceptance criteria row.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:99
    summary: Acceptance criteria leave two rules D1 and D2 introduce unpinned - D1's untimestamped transcript counted with the flag, and D2's dollar rule in a double-quoted span and on a special-parameter character
    fix: Under Acceptance criteria, add to the first -no-cutoff row that a main-session transcript with no timestamped user or assistant line is counted with the flag and left out without it when a bound exists; add `cat "${x@P}"` and `cat $@` to the false list of the IsReadOnlyCommand row.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D2 says every output redirection is rejected command-wide, but the classifier drops 2>&1 and 2>/dev/null before that scan
    fix: In D2 line 58, replace 'every output redirection' with 'every output redirection but the two dropped stderr forms `2>&1` and `2>/dev/null`'.
---

### [BLOCKING] ANSI-C quoting defeats the command-wide substitution bound

**Location:** `_lyx/discussion/decision-record.md:58` (D2), with Goal 2 at line 5 and the IsReadOnlyCommand criterion at line 105.

**Issue:** D2 bounds every other code-running vector in a reader segment by the command-wide checks, first among them the substitution markers found "with only single-quoted spans blanked".
That view comes from `blankQuotedSpans` in `internal/fabricengine/refscanner.go`, which knows no ANSI-C quote: in `$'…'` bash and zsh read `\'` as an escaped quote, but the scanner treats the `'` after the backslash as closing the span.
In `cat $'\'' $(touch x) f` the scanner closes at `\'`, opens an unterminated span at the next `'`, and blanks the rest of the command.
The view then holds no `$(`, no `>` and no `&`, the one segment starts with `cat`, D2's parenthesis check sees nothing in the blanked view, and D2's dollar rule admits the only visible `$` because a quote follows it.
`IsReadOnlyCommand` therefore returns true under D2, while bash runs the substitution: `bash -c "cat \$'\\'' \$(echo RAN >&2) /dev/null"` printed `RAN` here.
`lyx`, `git` and `sed` segments are safe, since `splitShellWords` rejects any `$`; only the reader path is open, and it is the path D2 changes.
So D2's stated bound is false, and Goal 2's "closes the two reader paths that can run code without a substitution marker" overstates what the change closes.
Scope Out forbids changing which commands are admitted beyond the parenthesis and dollar rules, so the fix belongs inside the dollar rule, where it fits: a `$` followed by `'` never forms a regex anchor in the blanked view, since a single-quoted `a$` is already blanked.

### [MEDIUM] The dollar rule enumerates bad followers and misses zsh's parameter flags and positional parameters

**Location:** `_lyx/discussion/decision-record.md:54` (D2).

**Issue:** The rule rejects `$` followed by `{`, `[`, a name character or `@ * # ? $ ! -`.
zsh's `$~x` turns `GLOB_SUBST` on for that one expansion, so `cat $~x` with `x` holding a qualifier glob runs code in a default zsh, the very vector D2 cites for `cat $x` under `GLOB_SUBST`; `~` is in neither listed set.
`$=x`, `$^x` and `$+x` are zsh parameter expansions with the same gap, though they do not glob.
"Name character" leaves open whether digits count, so `cat $1` is either rejected or admitted depending on the plan writer's reading, and under zsh with `GLOB_SUBST` a positional parameter holding a qualifier glob runs code.
D2's claim "Parameter and arithmetic expansions are rejected" is therefore false for zsh, the shell the parenthesis rule is written for.
A deny-by-default follower rule closes every such spelling, still admits `grep a$ f`, `grep 'a$' f` and `grep "^a$" f`, and also covers F1's `$'`.

### [LOW] Acceptance criteria miss two introduced rules

**Location:** `_lyx/discussion/decision-record.md:99` (Acceptance criteria).

**Issue:** D1 changes one counting rule beyond the bound: with the flag, a main-session transcript with no timestamped user or assistant line is counted, where it is left out today when a bound exists (`CountRun` in `tools/tokencount/count.go`).
No criterion pins it, so the plan can leave the change untested.
D2's dollar rule is applied in the view that keeps double-quoted spans, and that choice is what makes `"${x@P}"` non-read-only; no row has a double-quoted expansion, and no row has a special-parameter follower.
The rows present (`cat $x`, `${x@P}`, `${a[y]}`) pass equally under a rule that blanks double-quoted spans or omits the special-parameter set.

### [NIT] "Every output redirection" is not what the command-wide scan rejects

**Location:** `_lyx/discussion/decision-record.md:58` (D2).

**Issue:** `IsReadOnlyCommand` drops `2>&1` and `2>/dev/null` through `dropStderrRedirections` before the `>` scan, so those two forms pass.
They write nothing a reader could change, so the bound holds, but the sentence states a stronger check than the code runs.

## Notes on the focus questions

- D1 Bound: complete against what `-calibrate` reads.
  `readRunPlan` in `tools/tokencount/calibrate.go` takes the newest plan commit before `earliestFork(run.Forks)`, and D1 states that an earlier run's fork can move that point back, select an earlier plan, or yield none.
  `CountRun` skips the start check only for a non-zero bound, and D1 states that the zero bound counts untimestamped transcripts.
  `BuildProfileReport` matches forks by transcript file name against the newest webster record, so earlier forks do not enter batch figures beyond the plan-derived identity card counts D1 already names.
- Goal, Scope In, Scope Out and the Decision bodies agree on D1 and D3.
  For D2 they agree except where F1 notes Goal 2's overclaim, and F1's fix stays inside Scope Out's limit.
- D3's premise holds: every `loomyard-weft` in `CLAUDE.md` and `README.md` sits between brackets, slashes or parentheses, and `PATTERN.md` holds none.
