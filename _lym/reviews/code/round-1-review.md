---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/testcover/mutant.go:182
    summary: With every mutant off, the step rewrite changes the value of an untyped constant expression whose intermediate overflows the context type, such as (1 << 63) / 2 in a uint64 or int32 function
    fix: In fileSites in internal/testcover/mutant.go, add no OperatorChangeConstant site for a literal or package-level literal constant that is an operand, through parentheses, unary operators or nested binary operators, of a binary expression whose operands are all integer or rune constants; mark those nodes fixed as markFixed does. In the same commit, add a case to an existing PlanMutants or WriteOverlay test whose source holds func Half() uint64 { return (1 << 63) / 2 }, asserting that the plan holds no change-constant mutant on that line.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundantmutants.go:410
    summary: No test reaches round mutantBuildRounds, so reverting to the old failure at the cap, or never passing wholeFiles, fails no test
    fix: In internal/gatecli, add a case to the lym gate redundant tests that sets the request's goBinary to a stand-in go command. Model it on internal/testcover's testmain stand-in. Its test -c -overlay build refuses mutantBuildRounds builds in a row, each naming a line no remaining mutant spans in a mutated file, and then succeeds. Assert that the call finishes and reports the exact Unbuildable count that the whole-file drop at round mutantBuildRounds yields.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/testcover/mutant.go:116
    summary: The embedded-line rows cover neither the earlier of two separators winning nor the fallback to the indent when only blanks follow a matched separator
    fix: "In internal/testcover/mutantbuild_integration_test.go, extend embeddedLineText with a line holding = before a colon-space (for example a=b: c) and a line whose colon-space is followed by blanks only (for example key: with trailing spaces). Add one row each to TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine with the exact text expected, the mark after the = and the mark at the line's indent, and renumber the later rows."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/gatecli/redundantmutants.go:35
    summary: The mutantBuildRounds doc calls it the number of line-only builds, but round >= mutantBuildRounds with rounds counted from 1 makes build mutantBuildRounds itself drop whole files
    fix: Reword the doc comment of mutantBuildRounds in internal/gatecli/redundantmutants.go to say it is the build from which a refused build drops every mutant of the files the compiler names, and that the builds before it drop only the mutants of refused lines.
    origin: lens:code-correctness
---

### [MEDIUM] The step rewrite changes constant-expression arithmetic with every mutant off

**Location:** internal/testcover/mutant.go:182

**Issue:** `closing` writes each integer constant `c` as `(c + 1>>lymMutantStep(id))`, which makes the expression non-constant.
An untyped constant expression such as `(1 << 63) / 2` in a `uint64` function was evaluated exactly at compile time.
After the rewrite it is evaluated in the context type at run time: `uint64(1) << 64` is 0, so the function returns 0 even with no mutant on.
The binary compiles, so the drop rounds never see it.
The baseline run in `redundantmutants.go:196-197` then marks a candidate failing unmutated, or a covering test that kills nothing, and that evidence is lost.
The old generic helper defaulted to `int`, so this was already possible in an `int` context; the type-keeping rewrite extends it to every integer type.
The case is rare, and it loses evidence rather than listing a false candidate, so it is MEDIUM.

### [MEDIUM] The cap removal and whole-file switch in buildMutants have no test

**Location:** internal/gatecli/redundantmutants.go:410

**Issue:** `TestMutantPlan_UnbuildableMutants` passes `wholeFiles` directly.
`TestGateRedundant` (`redundant_integration_test.go:341`) reaches only a one-round line drop.
No test covers the caller's `round >= mutantBuildRounds` wiring, or that the loop no longer fails at the cap.
That is the second fix the record asks for: a mutant that does not compile never fails the whole check.
The assistant raised this too, as BLOCKING; it is an uncovered behaviour, not a constraint violation, so it is MEDIUM.

### [LOW] The separator precedence and blank-value fallback in lymMutantLine have no rows

**Location:** internal/testcover/mutant.go:116

**Issue:** `embeddedLineText` (`mutantbuild_integration_test.go:314`) holds one separator per value line.
Its bare key `list:` holds no `: `, so the branch that matches a separator and then falls back to the indent never runs.
The choice of the earlier of `: ` and `=` is not exercised either.
The commit message of `3a786793` states both rules.

### [LOW] mutantBuildRounds doc is off by one against the loop

**Location:** internal/gatecli/redundantmutants.go:35

**Issue:** The loop starts at `round := 1` and passes `round >= mutantBuildRounds`.
So builds 1 to 4 drop by line, and build 5 already drops whole files.
The constant's doc says five line-only builds.
The `buildMutants` doc and `internal/gatecli/doc.go` ("From build `mutantBuildRounds` on") agree with the code.

## Rejected

- Assistant F4, that discovery and the build disagree under `GOFLAGS=-tags=…`: the doc comment of `ScanTestFuncs` states this limit (a build tag set through GOFLAGS is not read), and `parsePackageFiles` already has the same one. The map builds pass no tags (`testmappackage.go:203`). The record asks about the host platform, and the gatecli doc's tier examples set no tags in GOFLAGS.
- code-correctness LOW, that a typed non-string value such as `port: ~8080` is still killed by every loading test: this extends the operator beyond what the record asks for. The diff's doc claims only cover unknown keys, which it handles, and all five false candidates are gone per the change description.
- code-tests NIT, the `MatchFile` error path with no row: it only passes up an OS open error for an unreadable file, and testing it adds no information about the behaviour.
- Assistant F1 to F3 severities: F2 and F3 merge into this review's F2 and F3, and F5 merges into F4. None is a constraint violation, so none is BLOCKING; the assistant's F1 is this review's F1, at MEDIUM for the reason given there.

Holistic notes: every hunk traces to a goal point or is forced by one, as the code-scope report shows.
The embedded-line change is needed to meet the Done-when, and dropping the tier1 tag follows PATTERN-test-speed's ban on tier tags in test-file build constraints.
The change description names the 2026-11-17 report in place of the 2026-11-12 pilot and says why; the pilot report holds no `boardengine` candidate.
That fault lies in the answer key, which the rubric forbids flagging.
The directive declares no PATTERN changes, and no commit carries a `Loosened-Check:` or `Pattern-Departure:` trailer.
