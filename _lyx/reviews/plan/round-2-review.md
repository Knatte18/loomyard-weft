---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/02-classifier-shell-words.md
    summary: readOnlySedSegment requires one or more file words, narrowing D2's sed form so a stdin print such as grep x f | sed -n 1,5p stays a correctness halt with no stated reason
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/08-thin-table-rows.md
    summary: the awaited-shell row needs an empty EndedShells and two absent log lines, which the planned defaulting EndedShells column and the table's single wantNoLog column cannot express as described
---

## Focus verdicts

**Focus 1, unjudged `git` words under the deleted `plainWord`: not a defect.**
In `internal/fabricengine/refscanner.go` the words `gitForm.admits` never sees are the `-C` directory (`readOnlyGitSegment`), the `cat-file` mode and object words, the `config` words, `branch --list` patterns (`readOnlyGitBranch`), the `branch` listing flags, `remote -v`, and the subcommand words themselves.
Card 2's Intent names the four that take free text: "`readOnlyGitSegment` rejects one in the `-C` directory and in the `cat-file` and `config` words, and `readOnlyGitBranch` rejects one in a `--list` pattern".
The others are matched exactly against closed sets (`listing`, the `cat-file` mode list, `-v`, the `gitForms` keys and the `remote` and `branch` subcommands), so an unquoted glob word in them never matches and the segment rejects without a glob check.
The `remote get-url` and `remote show` words go through `admits` with the new field unset, so a glob after `--` there rejects, as the Intent says.
The card states the bound ("an unquoted glob reaches git only after `--` in `log`, `show`, `diff` and `status`") and pins each free-text site with a rejected row: `git -C * status`, `git cat-file -p *`, `git config --get *` and `git branch --list *`.
I also checked zsh's `EXTENDED_GLOB` operators the card does not list: a mid-word `~` (exclusion) yields only names its literal left side matches, and a mid-word `^` after a `/` yields names prefixed by that path, so neither can produce a word starting with `-` unless the word already starts with one, which the flag rule judges.

**Focus 2, `TestWait_RecordsEndedShells` rows against unchanged production code: both rows pass.**
Ungated two-shell row: `pollEventsTick` records the waiting turn end, `expiredTurnEnd` finds every output file present, `countsAtOnce` returns true for an empty gate, and `countedTasks` takes both tasks, so `endedShells` returns both with their own signal and zero outstanding time, since the finish happens on the first tick before any sleep.
Awaited row: with a passing gate, `startedFresh` set and the label matching `AwaitedShellPrefixes`, `countsAtOnce` returns false at the `awaitedShell` check, so the turn keeps waiting.
The scripted `STOP:` line parses through `fakeEngine` to a non-waiting event, `pollEventsTick` clears `waitingTasks` and `countedTasks` and returns `OutcomeDone`, `handleGatedBoundary` passes and finalizes, and `endedShells` returns nothing, so `logEndedShells` logs neither line.
The row discriminates: an unawaited shell under the same setup counts at once and lands in `EndedShells`.
Production records an awaited shell whenever it is still outstanding at the end (an ungated at-once finish, the deadline, the liveness end), which the card states for the ungated case, and the card words the row's claim as holding "where the wait holds it"; that matches the code.
F2 covers how the row fits the table's columns.

**Focus 3, `TestIsReadOnlyCommand` rows and `PATTERN-test-economy`: not a defect.**
No planned row repeats an existing one: the closest pairs differ in the rule they reach (`git log "$X"` and `git log '$X'` against the existing unquoted `git log $X`; `git log '--output=x'` against the existing `git log --output=x`; `git -C * status` against the existing `git -C -c status`).
The three flipped rows ("descriptor redirection", "git log with a home path", "git log with a quote") appear only as flips, never again in the new lists, and `git log ~/x` from the decision record's admitted list is carried by the flip.
The decision record's `git log *.go`, `git log a?`, `git log [a]` and `git log {a,b}` are carried by existing rows, as the card says.
I found no existing row whose outcome changes under the new rules beyond the three the card names.

### [LOW] `sed -n` form requires a file word

**Location:** `_lyx/plan/02-classifier-shell-words.md`, the `readOnlySedSegment` sentence of the Intent.

**Issue:** D2's sed form is "`-n`, then one script word that is a line number or a range of two and `p`, then file words, none starting with `-`".
The card narrows it to "one or more file words", so `sed -n 1,5p` reading a pipe, as in `grep -n x f | sed -n 1,5p`, stays non-read-only and halts the batch as a correctness finding.
Reading stdin writes nothing and executes nothing, and none of D2's rationale (a trailing `-i`, a planted glob name, a `w` or `e` script) applies to zero file words.
The card gives no reason for the narrowing, and #518 is a halt over exactly this kind of harmless read.

**Fix:** in Card 2's Intent, change "then one or more file words" to "then zero or more file words", and add an admitted row `cat x | sed -n 1,5p` to the card's `TestIsReadOnlyCommand` rows.
If the narrowing is intended, state the reason in the Intent instead.

### [NIT] Awaited-shell row does not fit the table's columns as described

**Location:** `_lyx/plan/08-thin-table-rows.md`, the `TestWait_RecordsEndedShells` paragraph.

**Issue:** The card adds an expected-`EndedShells` column "defaulting to that shell's entry", and the awaited row expects `EndedShells` empty, so a zero value cannot mean both "default" and "empty".
The table also asserts `strings.Contains(log, wantLog)` and `!strings.Contains(log, wantNoLog)` with one string each, and an empty `wantNoLog` always fails since every string contains "".
The awaited row expects "neither ended-shell log line", which that shape cannot express.
The card's list of added columns (gate, started-fresh mark, awaited-shell prefixes, clock steps) names neither change.

**Fix:** in Card 8's Intent, say how the awaited row states an empty `EndedShells` (for example a `wantNoShells` mark) and that the log columns become a list of lines that must appear and a list of lines that must not, with the awaited row listing both ended-shell lines as absent.

## Prior-round check

Read after the findings above were saved.
Round 1's fixed F1, F2, F3 and F5 hold in the current cards, as the focus verdicts above show.
The disputed F4 stands disputed: `grep -hE '^func Test' cmd/lyx/*_test.go` without `TestMain` counts 72, equal to the `cmd/lyx: 72` budget row, so Card 4's `TestOwnJSONFlags` exceeds the row and the card has to raise it.
