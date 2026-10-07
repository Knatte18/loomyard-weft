---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/fabricengine/refscanner.go:215
    summary: IsReadOnlyCommand still classes a first word that only contains a quote as a reader, so cat"x" y (which runs catx) is read-only
  - id: F2
    severity: LOW
    class: scope
    location: internal/websterengine/recoverbatch_test.go:795
    summary: the recover-batch refusal test pins the committed read-only route but not the HEAD-at-start route its comment says it pins
  - id: F3
    severity: LOW
    class: scope
    location: internal/websterengine/acceptbatch_test.go:23
    summary: the entries that drive accept-audit --batch are hand-copied from the text fabricReferenceDetail writes, so nothing pins the parse in fabricReferenceCommand to the text the audit really records
---

Range: `status.json` gives `product.parent` `main`, and no `_lyx/loom/rework/` round exists, so the reviewed range is `git diff $(git merge-base main HEAD)..HEAD`, that is `35e36281e..HEAD`.
`go build ./...`, `go test ./...` and `lyx loom lint-comments --base 35e36281e` are green.
The diff implements cards 1 to 11; no card's intent is unmet.
The four focus questions are settled below the findings.

### [LOW] A first word that merely contains a quote still passes as a reader

**Location:** `internal/fabricengine/refscanner.go:215` (`commandWordQuoted`) and the segment loop at `refscanner.go:208`.
**Issue:** `IsReadOnlyCommand`'s doc says "A command whose first word is quoted is never read-only, since the quoted word is not a name the list can match."
`commandWordQuoted` only tests whether the first character of a command is inside a quote.
A word that opens with a reader name and then quotes the rest is read from the quote-blanked copy as the bare reader, so `cat"x" y` and `cat'' x` come out read-only, while the shell runs `catx` and `cat`.
I ran a copy of the three functions: `cat"x" y` → true, `cat'' x` → true, `ca""t x` → false, `\cat x` → false, `'tee' cat` → false, `"rm" cat x` → false, `ls; 'git' cat` → false, `\tee cat` → false.
The acceptance route's stated bound is "no listed reader can write"; a word spelled as a reader plus a quoted suffix names some other binary, so the bound does not hold for it.
The audit guards a well-meaning agent, not an adversary, so this is a hole in the stated guarantee and not an exploitable path, hence LOW.
**Fix:** Drop `commandWordQuoted` and look the first word up in its original spelling.
The segment boundaries come from the blanked copy, which has the same rune count as the original, so split the original command at the same rune offsets, take `strings.Fields` of each original segment, and require `readOnlyCommands[words[0]]` on that original word.
A first word carrying any quote or backslash then fails the lookup, which also makes the escaped-word case an explicit consequence instead of an accident.
Add a `TestIsReadOnlyCommand` row for `cat"x" y`, and reword the doc line to say a first word holding a quote or backslash is never read-only.

### [LOW] The recover-batch refusal test does not pin the HEAD-at-start route

**Location:** `internal/websterengine/recoverbatch_test.go:795`
**Issue:** The new constant's comment says "The route names both evidence: HEAD at the start commit, and a committed batch whose recorded commands are read-only", but `committedRouteText` pins only the second.
`acceptBatchStep` pins the two verb steps.
The wording `when the worktree is clean and HEAD is the batch's start commit (git reset --keep to it if the batch committed)` in `recoverbatch.go:336` is asserted nowhere, so dropping it, which the `audit: fabric-reference` row of `contracts/specs/refusal-spec.md` still requires, would pass.
**Fix:** Add the HEAD-at-start phrase to the `want` list in `fabricRefusalCheck`, in the same loop as `committedRouteText`; no new test.

### [LOW] Accept-route tests hand-copy the entry text instead of deriving it

**Location:** `internal/websterengine/acceptbatch_test.go:23`
**Issue:** `AcceptBatchFabricReference` recovers the command with `fabricReferenceCommand`, the inverse of the `%q` text `fabricReferenceDetail` writes, from entries `record-batch` stores as `<class>: <detail>`.
Every test that reaches the committed route builds its entry by hand (`readOnlyEntry` here, and the literal in `internal/webstercli/acceptaudit_test.go:115`).
Rewording `fabricReferenceDetail` would make every real committed batch refuse as "records no command", while all of these tests still pass.
**Fix:** Build `readOnlyEntry` from the real audit output: call `websterengine.CheckParent` with a `shuttleengine.ForkAudit` whose `ParentBashCommands` holds the command and a `RefMatcher` that matches it, and return `string(violation.Class) + ": " + violation.Detail`.
That reuses the existing rows and adds no test.

## Focus questions

1. **Quoted or escaped command word.**
   `'tee' cat`, `"rm" cat x` and `ls; 'git' cat` are all refused, and `TestIsReadOnlyCommand` carries rows for the single-quoted first word, a double-quoted word after `&&` and a quoted word after `|`.
   A `;` or newline separator takes the same `atCommandStart` branch as `&&` and `|`, so it needs no row of its own.
   A backslash-escaped word (`\tee cat`, `\cat x`) is refused because the escaped spelling is not a name in the list; no row pins it, and one is not needed once F1's fix makes that the lookup's own rule.
   The open part is a word that is only partly quoted, which is F1.
2. **recover-batch refusal against the `audit: fabric-reference` row.**
   The message names both routes: the worktree clean and HEAD at the start commit, or the batch's commits kept with every recorded command read-only, then the `accept-audit --batch` and `recover-batch` steps, and it reads as the row does.
   The test pins the committed-route phrase and the two steps but not the HEAD-at-start phrase, which is F2.
3. **`accept-audit --batch` usage line against `Long`.**
   The usage line says "once HEAD is at its start or its commands only read"; `Long` says the worktree is clean and either HEAD is the start, or the start is an ancestor of HEAD and every command is read-only.
   Both describe the same two routes; the usage line leaves out the clean-worktree and ancestor preconditions, as a one-line summary may.
   Not a defect.
4. **`tools/tokencount` rows.**
   The `n/a` growth ratio has a row in `TestCalibrate` (the flat-estimate case, whose expected row prints `n/a`, and `GrowthFitOf` over it).
   A malformed review file has the `garbled` run in `TestBuildProfileReport`, and a no-partition run with an unreadable plan has the `orphan` run, each asserting the printed mark.
   All three extend the existing tests; no gap.
