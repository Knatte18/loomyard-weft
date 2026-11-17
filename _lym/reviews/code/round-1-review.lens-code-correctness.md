```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff `44431fef..HEAD`.

**List: each function or branch, the case run, and the result**

- **`testcover.JudgeMutants`, the candidate's own run unjudged** (`unjudged[id][c.Name]`). Case: candidate `Stalls`, mutant 3 timed out, and covering test `B` killed 3 (the new row). Result: `Unjudged=[3]`, not killed or unique, and the candidate is listed unconfirmed. Correct as code, but see finding 1.
- **`JudgeMutants`, killed with a covering test's run unjudged and no covering test killing.** Case: `A` killed 1, `B`'s run of 1 timed out. Result: `Killed=[1]`, `Unjudged=[1]`, not redundant. Correct.
- **`JudgeMutants`, unique kill beside an unjudged mutant.** Case: `C` killed 1 and 2, `B` unjudged on 1 and absent on 2. Result: `Unique=[2]`, `Unjudged=[1]`, refuted (redundant.go checks `Unique&gt;0` before default). Correct.
- **`JudgeMutants`, nil `unjudged` map** (the `len(mutants)==0` path). Reading a nil map gives false. Correct.
- **`JudgeMutants`, no kills and no unjudged runs.** Result: `ReasonKillsNone`. Correct.
- **`JudgeMutants`, ordering.** Each id is visited once in sorted order, so `Unjudged` comes out ascending with no duplicates. Correct.
- **`JudgeMutants` greedy pass.** It reads only `kills` and only redundant candidates, which now never hold an unjudged mutant. Correct.
- **`runRedundant` switch default.** It is reachable only with `Unjudged` non-empty: no reason, not redundant and no unique together imply that. Correct.
- **`mutantSites`.** Ids come from candidate mutants, which are keys of `mutants`. Empty ids give an empty slice. Correct.
- **`runMutants` `record`.** A `TimedOut` or `MemoryCapped` result carries `ActionFail` and goes to its own map, never to `kills`. A plain fail goes to `kills`. A guard kill is skipped. Correct.
- **`runMutants` queue condition `!timedOut[run]`.** Case: candidate `X` covers killer `B`, and `X`'s candidate run of `id` timed out side by side. `X` is queued, its entry is cleared from `timedOut`, it reruns, and the alone result stands. A capped candidate run is not queued and stays unjudged. Correct.
- **`runMutants` rerun timing out again.** `runJobs(rerunFailed=true)` reruns a failed non-capped job alone. A timeout alone is then recorded in `timedOut` and becomes unjudged. Correct.
- **`runMutants` building `unjudged`.** It is built after the final `record`, from `timedOut` and `capped` after clearing. Correct.
- **`runMutants` error paths.** They return `nil, nil, err` and the caller propagates it. Correct.
- **`RunAlone` / `testOutput.Write`, output split mid-line.** A partial line stays unscanned until its `\n` arrives, then it is stamped at completion. Correct.
- **`testOutput.stamp`, subtests.** `=== RUN   X/sub` fails the equality check, and an indented `    --- PASS: X/sub` fails `HasPrefix("--- ")`. Correct.
- **`stamp`, parallel top-level test.** `=== PAUSE` and `=== CONT` are ignored, and the time runs RUN to result, including the pause. The pause is negligible when the test runs alone. Correct.
- **`stamp`, no RUN line.** `started` stays zero, so the first case swallows every line and `ended` is never set. `Elapsed` falls back to the binary's figure. Correct.
- **`stamp`, RUN line with no result line** (panic or crash). `ended` is zero, `Elapsed` is 0 from the regex and the action is fail. Correct.
- **`RunAlone` concurrency.** Stdout and Stderr are the same pointer, so exec serialises the writes. `cmd.Run` and WaitDelay wait for the copy goroutines, so `started` and `ended` are read after them. Correct.
- **`RunAlone` timeout and cap returns.** These come before any Elapsed handling. Unchanged. Correct.

**Findings**

1. **MEDIUM, design.** Location: `internal/gatecli/redundantmutants.go:287` (queue loop over `kills` only), with `internal/testcover/mutantverdict.go:82-84`.
   Summary: a mutant that the candidate's own run left unjudged is never run under the covering tests. The candidate is therefore unconfirmed even when a covering test would kill that mutant. In that case the candidate's confirmation does not rest on the unjudged run, which is the board entry's test.
   Before this change, a timeout counted as a kill, so the covering tests ran it. Now the queue iterates `kills` keys alone, and `JudgeMutants` sends every own-unjudged mutant to `Unjudged` without consulting the covering tests.
   Evidence: `redundantmutants.go:287-305`, `mutantverdict.go:82-85`, and the fixture comment at `redundant_integration_test.go:300-301` ("no covering test runs the mutants ending the loops").

No other correctness defect found.

Tool calls:
- `grep runTestBinary` / memory-cap Stdout: checked whether the memory-capped path writes Stdout and Stderr from separate goroutines, which would make `testOutput` race. The manifest lacked `runTestBinary`.
