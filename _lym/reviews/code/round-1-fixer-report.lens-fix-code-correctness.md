```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the changed code of the fix through concrete inputs and check callers and tests for correctness (lens fix-code-correctness).

**Findings**

1. **MEDIUM**, `conflictingResultsWord` in `internal/gatecli/trx.go`: it refuses `-r` as if it were `--results-directory`.
   - Evidence: the `case "--results-directory", "-r":` branch. In `dotnet test`, `-r` is the alias of `--runtime`, not of `--results-directory`.
   - Input: a call `lym gate test ... -- -r linux-x64` on a `results: trx` tier is refused with "the words after `--` hold -r, which the gate sets itself". That refusal is wrong, and its way forward drops a valid runtime option.
   - `-l` as the alias of `--logger` is correct. No test row covers `-r`, so the suite stays green.
   - Fix: drop `"-r"` from the case.

2. **LOW**, `runTierChecked` in `internal/gatecli/tierrun.go`: the F14 "print once" holds per attempt only.
   - Evidence: `retryTier` calls the attempt closure, which is `runTierChecked` at `internal/gatecli/run.go:631`, once more on a shortage or crash rerun. The env line is computed and printed on each attempt, so a rerun prints it twice.
   - The rerun's log also opens with the announcement and then the line, so the line shows once in each attempt's log. Only stderr repeats it.
   - No test covers this.

3. **LOW**, F12 (flush the counter before `replaceInvocation`): no test would fail with the defect in place.
   - Every fixture command prints with `echo`, so each console line ends in a newline. Deleting the `counter.Flush()` call before `readTRXResults` leaves `TestRunTierReadsTheVSTestResultsFile` green.
   - A row whose last console summary has no trailing newline (for example `printf` without `\n`) would pin it.

4. **LOW**, F10 refusal: the joined forms of `--logger` and `--results-directory`, such as `--logger:trx` and `--results-directory=x`, plus `-l trx`, are handled in code but untested. Only the two-word spellings have rows.

**Checked, no defect**
- `runInvocations`:
  - the results options are injected only when `Results == ResultsTRX`;
  - `counter.(*vstestCounter)` cannot panic for a loaded config, because `decodeTier` requires `counter: vstest`;
  - the refusal fires before the prebuild and before any command starts;
  - the results directory is removed on every path.
- `MeasureCall`:
  - its only production caller is `darnwiring.go`, which now propagates the error;
  - a hub with no tiers still returns the no-target call;
  - the card-scope and no-`{args}` refusals match `planTierRuns` and `gateKeyRefusal`.
- `trx.go`: the Error-only filter maps to the TRX `RunInfo outcome` attribute, and the test host crash fixture uses `outcome="Error"`.
- New tests that would fail with the defect in place:
  - the two-invocation row fails without the `startInvocation` baseline;
  - the outcome-mapping row fails if Error, Timeout or Aborted were counted as skipped;
  - the command-args assertion fails if `--logger` or `--blame` were dropped;
  - the Warning `RunInfo` in the crash row fails with the old all-messages decode;
  - the `COPIES` row fails if each copy printed the line;
  - the "session's value in neither stderr nor log" assertion fails with the old line;
  - the integration case fails if `request.sessionEnv = os.Environ()` is removed.

No further tool calls were made; the context already held every changed file.
