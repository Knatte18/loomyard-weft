---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/checkout.go:137
    summary: switchOrForkWeft doc keeps the independent clause "so rollback can delete it" on the line of the clause before it
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/remoteleftover.go:2
    summary: header line 2 and the leftoverKind doc line 20 each hold an independent clause joined by a comma plus so or and with no line break
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/deps.go:126
    summary: ConfigChanges field doc splits one compound predicate across lines 126 and 127, and Notify field doc line 138 keeps an independent clause after comma plus so on one line
---

The build, `go vet -tags integration` over the five touched packages, and the untagged tests of `fabricengine`, `fabriccli`, `landingshed`, `loomcli` and `pattern` all pass.
The sweep covered every `//` line the range `git diff $(git merge-base main HEAD)..HEAD` adds or changes in the four named packages plus `hubgeom`, test files included, against `PATTERN-comment-line-breaks`.
The three lines at `add.go` 302-304 (a break inside a clause) are legacy text re-indented, not changed text, and are not findings.
No other finding was raised: the diff's behavior matches the plan's cards as far as the comment sweep reached, and no new test duplicates an existing one that I found.

### [BLOCKING] switchOrForkWeft doc line 137 joins two independent clauses

**Location:** internal/fabricengine/checkout.go:137
**Issue:** `// It reports whether this call created the local branch (adopted from origin or forked), so rollback can delete it;` holds a clause and a comma-plus-`so` clause with its own subject and verb (`rollback can delete it`) on one line.
**Fix:** Break before `so rollback can delete it;`, keeping the trailing `;`.

### [BLOCKING] remoteleftover.go header line 2 and leftoverKind doc line 20 hold two clauses on one line

**Location:** internal/fabricengine/remoteleftover.go:2 and internal/fabricengine/remoteleftover.go:20
**Issue:** Line 2, `A branch on the weft or warp origin either makes the pair live, so Add adopts it, or is a removed pair's leftover that is proven replaceable or refused before Add's first mutation,` carries the independent clause `Add adopts it` after comma plus `so`, inside an either/or that a break cannot split cleanly.
Line 20, `A remote weft branch no archive tag covers is not one of them: it makes the pair live, and Add adopts it.` joins `it makes the pair live` and `Add adopts it` with comma plus `and` and no break.
**Fix:** Restructure the header so each clause has its own line: state that a branch makes the pair live or is a removed pair's leftover, then `Add adopts a live pair's branch;`, then `a leftover is proven replaceable or refused before Add's first mutation,` and `so the refusal never arrives mid-Add as a rejected push.`
On line 20, break before `and Add adopts it.`

### [BLOCKING] Deps.ConfigChanges and Deps.Notify field docs break a compound predicate and keep a clause joined

**Location:** internal/landingshed/deps.go:126 and internal/landingshed/deps.go:138
**Issue:** Lines 126-127 are `Finalize calls it before the catch-up merge-in, so before any parent-side mutation,` / `and only reports the result through Notify:`, which breaks between the two predicates of one compound predicate (`calls it ... and only reports ...`).
The sibling doc at line 136 keeps the same shape on one line.
Line 138, `landingshed names no orchestrator path, so the queue wiring lives in the closure.` joins two independent clauses with comma plus `so` and no break.
**Fix:** Put the compound predicate of lines 126-127 on one line, `Finalize calls it before the catch-up merge-in, so before any parent-side mutation, and only reports the result through Notify:`, or rewrite it so the `so` fragment follows the second predicate.
On line 138, break before `so the queue wiring lives in the closure.`
