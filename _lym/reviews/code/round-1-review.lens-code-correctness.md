```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each changed function and branch through concrete inputs.

**Functions and branches, with the case run and the outcome**

- `vstestResultsArgs("/t/r1")`: returns `--results-directory /t/r1 --logger trx;LogFilePrefix=gate --blame`. The words are quoted by `quoteAll`. Correct.
- `readTRXResults`:
  - A missing directory warns and returns false, so the console counts stand. Correct.
  - An empty directory returns false. Correct.
  - One malformed `.trx` beside one valid one: the bad file is left out and read is true. Correct.
  - One file per project: the counts are summed. Correct.
  - Blame sequence files in a GUID subdirectory are skipped by the `IsDir` check. Correct.
- `parseTRXFile` on a TRX with the 2010 `xmlns`: unprefixed Go xml tags match any namespace. Correct.
- `trxResults.add`:
  - Outcomes `Passed`, `Failed`, `NotExecuted` and `Inconclusive` map to passed, failed, skipped and skipped. Correct.
  - Run messages: every `RunInfo` `Text` is taken, whatever its `outcome`. See F2.
- `firstLines`: limit 0 returns nil, limit -1 returns all lines, and blank lines are dropped. Correct.
- `startInvocation`, then a console summary of 1/1, then `replaceInvocation` with a TRX of 2 discovered, 1 failed: counts become 0+2, and failures are cut to the marked length before the TRX failures are appended. Correct.
  - Two invocations: the second's mark is the first's replaced counts. Correct.
- `replaceInvocation` is skipped when no TRX was read, so the console counts stay. Correct, as the doc states.
- `RunMessages` returns a clone. Correct.
- `runInvocations`:
  - `MkdirTemp` fails: a warning is logged, `startInvocation` is not called, and the run falls back to the console counts. Correct.
  - Render error after `MkdirTemp`: the directory is removed, then the refusal returns. Correct.
  - Start failure (`cmd.Process == nil`): the directory is removed before that check. Correct.
  - Memory-cap refusal: the directory is removed before it. Correct.
  - Timeout or cancel: the command is killed, the TRX is read if present, and the directory is removed. Correct.
  - Copies: each copy gets its own counter and its own temp directory. Correct.
  - Retry: each attempt gets a fresh counter. Correct.
- `noTestNotice` after a TRX replace: `discoveredBefore` is read after `startInvocation`, so the delta is the TRX's discovered count. Correct.
- `runTierCommands`:
  - With exit 0, no run message is printed. Correct.
  - With exit 1 and a crash `RunInfo`, the line goes to the log, stderr and the causes. Correct.
  - With exit 1 and a failing test plus a warning `RunInfo`, the warning becomes a cause. See F2.
- `envOverrideLine`:
  - The session has `A=1` and the tier `A=2`: the line is printed. Correct.
  - Values equal, or the variable unset in the session: no line. Correct.
  - A session entry without `=` is skipped. Correct.
  - Duplicate session keys: the last one wins, as exec resolves them. Correct.
  - `GOFLAGS`: the printed value lacks the `-count=1` that `WithTestCacheOff` prepends. See F5.
  - Copies run: see F4.
- `request.sessionEnv = os.Environ()` in `runTest`:
  - A detached runner inherits its starter's environment, minus only the strand name (`detachfront.go:171`). Correct for the session that starts the run.
  - `RunGate` goes through `runTest`. Correct.
- `removeResultsDir("")` is a no-op; a failed `RemoveAll` only warns. Correct.
- `Config.Gate` with an unknown key: the refusal lists the keys sorted, or says the config holds no key. Correct.
- `Config.MeasureCall`:
  - Zero config: `--tier 0 -- <args>`. The gate refuses this at run time because no `gate.yaml` exists, but no form could run there. Acceptable.
  - Key with a tier that holds `{targets}`: the target word is added. Correct.
  - Mixed tiers: a tier without targets runs whole with a notice, not a refusal (`tiers.go:239`). Correct.
  - Hub with tiers but no `tier0`: see F3.
  - Key of scope `card` with no tier that takes targets: see F3.
  - Hub with no tier: `--gate k -- <args>`. Correct.
- `wireDarn`:
  - `ErrConfigAbsent` gives the zero config, so the table still builds. Correct.
  - Any other load error is returned, and Preflight wraps it with its own way forward. Correct.
- `DarnTable`: an empty `MeasureCall` is refused by name. The ungated fragment holds no marker, so the filled value is unused there. Correct.
- `makeChangeList` fills both markers. Correct.

**Findings**

1. **F1. LOW, design, `internal/gatecli/tierrun.go:485-495`.** The gate's results args are prepended to the call's own words without checking them for `--results-directory` or a trx `--logger`.
   - If `dotnet test` lets the user's later `--results-directory` win, or rejects the duplicate option, the gate's directory gets no TRX file. The console counts then stand again, and a detailed console logger can again turn the verdict into "ran no test".
   - This contradicts `doc.go`'s "A console logger among the call's own words so never changes the verdict".
   - Evidence: `tierrun.go:492` (`args = append(vstestResultsArgs(dir), run.args...)`) and `trx.go:521-543` (the console fallback when no file is read).
   - I could not check how `dotnet test` itself resolves the duplicate here.

2. **F2. LOW, design, `internal/gatecli/trx.go:505` and `:572-576`.** `trxFile.RunMessages` takes every `RunInfo` `Text`, whatever its `outcome` attribute.
   - A failed run therefore lists warning-level run messages as failure causes, on stderr and in the log. An example is "No test matches the given testcase filter …", including one from an earlier invocation that passed.
   - The doc says each such message is a test host crash.
   - Evidence: `trx.go:505` and `:572`; `tierrun.go:152-158`; `doc.go` "Such a message is a test host crash with the test the blame collector saw running."

3. **F3. LOW, design, `internal/gateslot/tiers.go:436-448`.** `MeasureCall` can render a call the hub's config refuses, against its own doc's claim "so the call names only a form the hub's gate config can run". Two cases:
   - **No `tier0`.** When `Gate` refuses because the hub declares tiers but no `tier0`, the fallback `lym gate test --tier 0 …` names an undeclared tier, so the gate refuses it too.
   - **Scope `card`.** When the key has scope `card` and none of its tiers takes targets, `Gate` succeeds. The rendered `lym gate test --gate <key> -- <args>` carries no `--card` and no target, so `gateKeyRefusal` refuses it.
   - Evidence: `tiers.go:393-394`, `tiers.go:439-447` and `run.go:450-451`.

4. **F4. NIT, design, `internal/gatecli/tierrun.go:138-142`.** Under `--copies N`, every copy's `runTierCommands` prints the env line, so stderr carries the same line N times.
   - Evidence: `copies.go:90-95` calls `runTierCommands` per copy with the shared locked stderr.

5. **F5. NIT, design, `internal/gatecli/tierrun.go:139`.** The env line compares and prints the tier's raw `GOFLAGS`. The child really gets `gateslot.WithTestCacheOff(tierEnv)`, which prepends `-count=1`, so the printed "runs with" value is not the one the tier ran with.
   - Also, on Windows the name comparison is case-sensitive, but environment variable names there are not.
   - Evidence: `tierrun.go:139` and `:143`; `envOverrideLine`'s `sessionValues[name]` lookup.

**Unverified risk, not filed.** `trxLogFilePrefix`'s doc says the target framework plus a timestamp make files unique per project. If VSTest's prefix timestamp has only seconds, two same-framework projects finishing in the same second would write the same name, and the counts would drop silently. I could not check VSTest's filename format here.

**Tool calls**

- `grep` in `internal/gatecli` for the detached runner and front files: to learn whether the runner's `os.Environ()` is the calling session's. The manifest did not cover `detachfront.go`.
- `grep` of `detachfront.go` for its env handling and joins: it confirmed the runner inherits the front's environment (`detachfront.go:171`).
