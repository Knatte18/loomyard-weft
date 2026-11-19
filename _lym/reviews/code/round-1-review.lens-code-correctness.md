```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function and branch the diff adds or changes, trace one concrete input step by step.

## Per function or branch: the case I ran, and the result

- **`spreadEvenly`**
  - n=4 over 8 open groups picks indexes 0, 2, 4, 6. That gives `a` IDs {1,5} and `b` IDs {1,3}, matching the test row. Correct.
  - n ≥ len returns every item. Correct.
  - n=0 cannot reach it, because the loop needs `taken < maxMutants`, and config requires `mutation_max_mutants` ≥ 1. Correct.
  - Indexes are distinct whenever len > n. Correct.
- **`selectWithinCap`**
  - Cap 10 over 8 functions: round 0 takes 8. Round 1 has 4 open groups and n=2, so it takes `a`/F0 and `a`/F2, which are IDs 2 and 6. Not run = 12−10 = 2. Correct.
  - Groups are sorted after map iteration, so the result is deterministic. Correct.
- **`logsOnly` / `blockLogsOnly`**
  - `if n > 10 { logger.Info(...) }` is noise, so its condition gets no mutant. Correct.
  - An `if` that logs and also returns is not noise. Correct.
  - An empty body is not noise. Correct.
  - An `if` with an init statement is not noise. Correct.
  - An `else` block, or an `else if`, is recursed into. Correct.
  - A logger name shadowed by a local is not noise (`calleePackage` returns ""). Correct.
- **`comparesOtherPlatform`**
  - `runtime.GOOS == "plan9"` on Linux is noise, and the condition span it covers is dropped. Correct.
  - `"plan9" == runtime.GOOS` (reversed operands) is noise. Correct.
  - A comparison with this platform's name, through `!=`, is kept. Correct.
  - `<` is not considered. Correct.
  - `x.GOOS` on a non-runtime import is not noise. Correct.
  - In `a && runtime.GOOS=="windows"`, only that operand is noise. This matches the summary. Correct.
- **Noise containment loop / `mutatesBehavior`**
  - The site span is checked for containment in the noise span. The drop-statement site of a logger `ExprStmt` has the same span as its noise span, so it is still dropped, which keeps the old rule's behaviour. Correct.
- **`mutationCall.run`**
  - Each of `ChangedPaths`, `changedFuncsAt` and `changedLinesAt` failing gives an error wrapped with a way forward. Correct.
  - Zero changed functions gives an empty result with no error. Correct.
  - Cancellation after the refresh returns `ctx.Err`, and `runMutate` maps it to its cancelled message. Correct.
  - A failing `check.run` returns an error. In the verb this is correct. **In the report step it is not: see F1.**
- **`runMutate` refactor**
  - The messages are the same as before: "gate mutate: "+err. The cancel paths are kept. Correct.
- **`fillMutateAnswer`**
  - `notJudged` is non-nil, so the JSON lists stay lists. Correct.
- **`mutationStep.run`**
  - The red, out-of-hub and non-go-scoper branches each write a Note report with the gate key and time. Correct.
  - A write failure logs at Warn and prints no line. Correct.
- **`mutationStep.answer`**
  - Failures in the module, fabric or prime lookups give Errors with no Functions. Correct.
  - No parent branch gives `noteNoParent`. Correct.
  - A `check.run` or slot error: **wrong, see F1.**
  - A `goFileHashes` failure drops that package from `FileHashes`, so a later match skips its file check. This is minor and is the same pattern as the reach step. Not filed.
- **`MutationReportMatches` / `changedFunctionsMatch`**
  - The refactor carries `ModuleDir` and `FileHashes` over unchanged. Notes, red-gate reports and prime reports never match. Correct.
- **`SummaryLine` / `counts`**
  - A Note report prints its note. A judged report prints the six counts, plus ", N errors" when it holds errors. Correct.
- **`ReviewLines`**
  - 45 survivors give 46 lines. These are cut to 39 plus the cut line, 40 in total. Correct.
- **`renderMutationReportNote` / `mutationReportNote`**
  - Missing, red, Note, prime, read-error, match-error, mismatch and match each render the expected line. Correct.
- **`run.go` mutation branch**
  - A key that lists only `mutation` starts the guard and runs the step. Correct.
- **`decodeGates`**
  - `mutation` is accepted, and an unknown report names all three values. Correct.

## Findings

**F1: MEDIUM, class `design`**
- **Location:** `internal/gatecli/mutation.go:85-111` (mutationStep.answer). Evidence is from the `mutationreport.go` hunk at diff lines 490-496 and `internal/gatecli/mutatecheck.go:177-206`.
- **Summary:** A report whose mutation check failed or was cancelled is written with no Note, but with the changed functions and file hashes recorded. It therefore matches HEAD, and it shows zero counts after the partial results are thrown away.
- **What happens:**
  - `diffMutation.run` returns `mutateOutcome{}, err` when the slot hold fails, a package run fails or the context is cancelled. Everything judged so far is lost, and `found.outcome` stays empty.
  - `answer()` still fills `report.Functions` and `report.FileHashes` from `found.funcs`, and it sets no `Note`.
  - `MutationReportMatches` then returns true. Code-Review's `mutation_report` note reads "matches HEAD's tree" with "0 mutants run, … 0 survived, …, 1 errors".
- **Contradiction:** `MutationReportMatches`'s doc says "A report that judged nothing … never match[es]". The reach step's report recorded no functions on its error paths, so it never matched.
- **Fix:**
  - Leave `Functions` unrecorded (or set a Note) when `mutationCall.run` returns an error, so the report cannot match.
  - Or keep the partial outcome that `diffMutation` gathered before the failure.

## Tool calls

1. `sed -n 150,245p internal/gatecli/mutatecheck.go`: read `diffMutation.run`'s error returns, to see whether an error throws away the partial outcome. My context held only `selectWithinCap` and the renamed types.
