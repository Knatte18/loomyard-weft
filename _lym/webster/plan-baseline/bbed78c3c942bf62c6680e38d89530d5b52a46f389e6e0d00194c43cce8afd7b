# Card 8 — kwinplace-script-runner

**Create:**
- `internal/kwinplace/contract_test.go`
- `internal/kwinplace#Runner`
- `internal/kwinplace#NewRunner`
- `internal/kwinplace#Runner.Query`
- `internal/kwinplace#Runner.Apply`
- `internal/kwinplace#Runner.ReadTilingOutput`
- `internal/kwinplace#Runner.run`
- `internal/kwinplace#ErrScriptTimeout`
- `internal/kwinplace#LeftoverScriptError`
- `internal/kwinplace#acquireWithin`
- `internal/kwinplace#recordedExec`
- `internal/kwinplace#TestRunner_ScriptRuns`
- `internal/kwinplace#TestRunner_OverlappingRunsTakeTheScriptLockInTurn`
- `internal/kwinplace#TestAcquireWithin`
- `internal/kwinplace#TestContract_TheRealKWinAnswersAsRecorded`
- `internal/kwinplace#TestContract_TheRealKWinAppliesAsRecorded`

**Edit:**
- `//pattern/PATTERN-test-speed.md`
- `//docs/benchmarks/running-tests.md`

**Extends:**
- `internal/kwinplace/doc.go`

**Uses:**
- `internal/kwinplace/testdata/recordings/neve-query/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-kscreen/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-qdbus/load.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/load-taken.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/loaded-true.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/loaded-false.txt`
- `internal/standalonestate#UserStateDir`
- `internal/lymtemp/lymtemp.go#`
- `internal/state/state.go#`

**Intent:** Run D3's scripts against KWin under the per-user script lock, reading each report from the journal (D3, D10's query sharing).

`Runner` holds the per-user state directory, the exec seam (`func(ctx, name string, args ...string) ([]byte, error)`),
a clock, a sleeper, a run-id source and the bounds below as fields, so a test shortens them (`PATTERN-test-speed`).
One of the bounds is the per-call timeout, 5 seconds in production:
every call through the exec seam runs under a context bounded by it, and a call that passes it fails the run.
`NewRunner` fills them for production: `standalonestate.UserStateDir()`,
an exec that logs each spawn at Debug and its end through `internal/logger`, the real clock, and a random run id.
The file names `kwinscript.lock`, `kwinscript.loaded` and `kwinquery.json` are named constants joined onto that directory;
card 11 adds `kwinplace.lock` beside them.

`run` is one script run, in this order:
1. Take `kwinscript.lock` through `acquireWithin` with `lock.AcquireWriteLock`, for at most 30 seconds and at most `within`;
   a run that cannot take it in time fails before its load.
2. Read `kwinscript.loaded`; a name there is unloaded and waited out as in step 7.
   A name still loaded after that wait fails the run before its load with a `LeftoverScriptError`, and stays in the file.
   The error names the plugin name and its way forward:
   `qdbus6 org.kde.KWin /Scripting unloadScript <name>`, then the next placement runs again.
3. Write its own plugin name, `lym-kwinplace-<run id>`, to `kwinscript.loaded`, and the script to a temp file through `lymtemp.CreateTemp`.
4. Note the time, then `qdbus6 org.kde.KWin /Scripting loadScript <file> <plugin>`; an answer of `-1` fails the run.
5. `qdbus6 org.kde.KWin /Scripting/Script<id> run`.
6. Read `journalctl --user -u plasma-kwin_wayland.service --since <time> -o cat` through `ReportLines` until the end marker shows,
   at most four times a second for at most 5 seconds; no end marker in time fails with `ErrScriptTimeout`.
7. On every exit after the load call, the end marker, the timeout, a call past the per-call timeout and any error,
   `qdbus6 org.kde.KWin /Scripting unloadScript <plugin>`, then read `isScriptLoaded` four times a second for at most 2 seconds.
   Once the name reads as not loaded it empties `kwinscript.loaded`; a name still loaded stays there for the next holder.
   It releases the script lock either way.
A cancelled `ctx` still unloads and releases, under a context of its own bounded by step 7's wait, so a child ended by SIGTERM leaves no script it can remove.
Every bound is a bounded wait under a minute, outside `PATTERN-daemon-wakeups`' list.

`acquireWithin` starts `acquire`, the blocking acquire, once, in a goroutine, and returns when it lands, when `bound` passes or when `ctx` ends.
An acquire abandoned past its bound releases the lock at once should it land later, so the caller goes on without holding it.
Production passes `lock.AcquireWriteLock`, which wakes only when the lock frees, so a wait on a held lock makes no attempts (D10).

`Query` runs `QueryScript` and parses its report, and writes the snapshot with its end time to `kwinquery.json` through `state.WriteJSON`.
Under the script lock it first reads `kwinquery.json`: a report for the same tiling output that ended after `after` is returned with no script loaded,
so one query serves every child waiting at once (D10).
A `no-output` report returns `ErrNoTilingOutput`, naming the output.
`Apply` runs `ApplyScript` and parses its report.
`ReadTilingOutput` runs `kscreen-doctor -j` through the exec seam and reads it with `TilingOutput`.

`recordedExec`, in a `_test.go` file, is the one stand-in for `qdbus6`, `journalctl` and `kscreen-doctor` (`PATTERN-test-stand-ins`).
It answers each call with a recording picked by what the call sent, and logs the calls for assertions.
Card 1's `neve-qdbus` recordings answer the `qdbus6` calls:
a load's id, a load of a plugin name already loaded (`-1`), and `isScriptLoaded` reading true and false.
A test's run-id source returns the run id the journal recording carries.

Tests, `tier0` (`recordedExec` spawns nothing; the locks are files under `t.TempDir()`):
- `TestRunner_ScriptRuns` rows:
  - a query reads its snapshot, unloads its plugin and empties `kwinscript.loaded`;
  - a load answered `-1` fails the run and unloads by plugin name;
  - a load call answered only after the per-call timeout fails the run, unloads by plugin name and releases the script lock;
  - a plugin name left in `kwinscript.loaded` by a killed run is unloaded before the next load;
  - a name that still reads as loaded after the 2-second wait is left in `kwinscript.loaded` and the lock is released,
    and the next run fails before its load with a `LeftoverScriptError` whose text names the plugin and the unload command, while that name stays loaded;
  - a journal cut before its end marker fails with `ErrScriptTimeout` and unloads its plugin;
  - a cancelled context still unloads and releases;
  - a script lock another holder keeps past the 30-second bound fails the run with no load call;
    a `within` shorter than that bound fails it at `within`;
  - a query that finds a report in `kwinquery.json` newer than its `after` loads no script, and one older than it loads one;
  - `ReadTilingOutput` over the kscreen recording names Neve's priority-1 output.
- `TestRunner_OverlappingRunsTakeTheScriptLockInTurn`: a query and an apply on two runners sharing one state directory, started together,
  hold the script lock one after the other (the exec log shows no load between the other's load and unload),
  each under its own plugin name and each reading its own end marker.
- `TestAcquireWithin` rows, over real locks with a wrapper that counts calls and passes each to `lock.AcquireWriteLock`:
  with the lock held, `acquireWithin` returns not-taken at its bound, having called `acquire` once;
  once the holder releases, a third acquire takes the lock, so the abandoned one released it;
  with the lock held and then released within the bound, `acquireWithin` takes it, still with one call.
Together they add under 1 second to one `tier0` run.

`internal/kwinplace/contract_test.go` is `tier3`, the by-hand tier, so each of its runs is the operator's call.
It holds `TestContract_TheRealKWinAnswersAsRecorded` and `TestContract_TheRealKWinAppliesAsRecorded`; neither has a skip or a flag of its own.
With `NewRunner` the first runs a real `Query` and `ReadTilingOutput`, read-only, and compares what lym reads with the recordings' shape:
the report's kinds, the snapshot's fields present, the tiling output among the query's outputs, and the end marker read; never free text.
It then runs steps that are read-only or revert themselves, each compared with its recording by answer or by kind and fields:
- a second load of a plugin name still loaded, compared with `load-taken.txt`;
- `isScriptLoaded` before and after that script's unload, compared with `loaded-true.txt` and `loaded-false.txt`.
`TestContract_TheRealKWinAppliesAsRecorded` runs an `Apply` that places a throwaway window on a desktop that is not current,
compared by kind and fields with `neve-apply`.
Its doc comment states that it moves a throwaway window, so the operator runs or approves it by name:
`lym gate test --tier 3 ./internal/kwinplace -- -run TestContract_TheRealKWinAppliesAsRecorded`.
`-run TestContract_TheRealKWinAnswersAsRecorded` runs the steps that move nothing;
a `tier3` run of the whole package runs both tests, so it too moves a window.
`pattern/PATTERN-test-speed.md`'s tier table gains, on its `tier3` row, a real desktop session beside a real LLM session, both run by hand.
`docs/benchmarks/running-tests.md`'s `tier3` lines, the tier list's entry and the command block's comment, name a real desktop session beside the LLM session.
`doc.go` gains a paragraph on the script lock and the journal channel.

**ImpactSummary:** New code in the new package only; `doc.go`, the test-speed background's tier table and the running-tests page's two `tier3` lines each gain a desktop session, and nothing outside kwinplace calls the runner until card 12.
