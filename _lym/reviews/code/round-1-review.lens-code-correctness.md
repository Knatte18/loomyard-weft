```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each function or branch the diff adds or changes against one concrete case.

## List: function or branch, case run, verdict

- `buildLaunchCmd` (`command.go:52`): empty writable roots, empty model and empty effort each drop their `-c` or `-m` part. A nonempty set quotes correctly. `sh.WithEnv` gives `CODEX_HOME='..' codex ...`, and `Chain` joins with `;`, so `exit` runs after any Codex exit status. Correct.
- `buildResumeCmd` (`command.go:73`): returns `exit`. Correct.
- `tomlString` (`command.go:137`): `"`, `\`, `\n`, `\t`, `\r` (→ `\u000D`) and 0x7f (→ `\u007F`) all escape validly. Correct.
- `recordingCommand` (`command.go:101`): the raw-string `\n` in the `printf` format reaches the shell as `\n`. The `%s` is the only conversion, and the date comes in as an argument. A failed `date` leaves an empty time, which the parser reads as zero. Correct.
- `activityCommand` and `hookSetting` (`command.go:109`, `127`): matcher and no-matcher forms are well formed. Correct on Linux. `path.Dir` on a `filepath`-built path would break on Windows, which the Sketch puts out of scope.
- `validateSpec` and `validateEffort` (`codexengine.go:56`, `config.go:125`): an empty effort passes, an unlisted effort is refused naming the key, and a nil `Efforts` refuses every nonempty effort. Correct.
- `expandHomePath` (`config.go:110`): `"~/x"` expands. `"~"` alone, `""` and a relative value are refused by key. Correct.
- `Prepare` ordering (`codexengine.go:78`): config, spec, homes, anchor, checkout, folders and `writableRoots` all run before `prepareHome`. The rules refusal and `writableRoots` errors therefore leave no artifact. Correct.
- `readCheckout` (`home.go:57`): a plain clone gives an empty `mainRoot`. A linked worktree gives `Dir(CommonDir)`. A common dir not named `.git` gives no main root. `mainRoot == root` is dropped. An anchor outside a repository gives a wrapped error. Correct.
- `checkCodexFolders` (`home.go:83`): rules in the worktree or anchor refuse and log a Warn. The worktree config warns. The main checkout's config warns independently of the worktree's own. Correct. It also refuses on the anchor's `.codex/rules`, which is slightly wider than the Sketch's "worktree alone" and is conservative.
- `writableRoots` (`home.go:109`): `.lym` and `_lym` both links to `<weft>/.lym` and `<weft>/_lym` give one deduplicated root `<weft>`. Real directories give none. A dangling link makes `PointsTo` fail and Prepare refuse, which is acceptable. Correct for the layout the engine assumes.
- `prepareHome` and `ensureLink` (`home.go:134`, `159`):
  - A missing link is created.
  - A matching link is kept.
  - A regular file or a link elsewhere is kept with a Warn.
  - A create race re-checks `RawTarget`.
  - On Windows the `auth.json` refusal comes before any write, since the parent is created only inside `CreateFileLink` after the Windows early refuse.
  - Correct.
- `UpdateJSON` closure (`home.go:142`): `config.toml` is written before the record. If the record write then fails, `config.toml` trusts a root the record lacks. The next Prepare rebuilds both, so this is harmless.
- `mergeRoots` and `renderConfigTOML` (`home.go:183`, `204`): recorded roots whose folder is gone are dropped, the union is sorted, and an empty list gives a header and `[tui]` only. Correct.
- `CheckAfterStartup` (`home.go:221`): an empty anchor, a missing `config.toml` and a config load error each end in a Warn and never fail. A missing root warns once per root. Correct. `run.go:746` calls it after `awaitStartup` succeeds and before the colour and prompt steps, so a failed startup skips it.
- `ParseEvents` (`events.go:21`): stamp lines, malformed lines and non-`Stop` payloads yield nothing. A `Stop` payload yields one event. Correct.
- `ParseSessionSignals` (`events.go:55`):
  - A partial last line stops at the line start.
  - A trailing run of stamp lines holds `consumed` back to the earliest stamp.
  - A stamp's time pairs with the nearest preceding untaken stamp of the same hook.
  - Correct for sequential hooks. Finding 2 below covers interleaving.
- `typedInstruction`, `sessionStartSource` (`events.go:143`, `152`): `/clear` gives no instruction, `/tmp/x` is an instruction, `startup` and `resume` map and anything else maps to empty. Correct.
- `readRolloutEvents` (`rollout.go:37`): a partial last line is skipped, and `info: null` leaves `hasContext` false. `ContextTokens` filters on `token_count` and reads the newest. Correct.
- `TurnStartInterrupt`: see finding 1.
- Pane readings (`pane.go`):
  - `Startup`: the trust prompt comes first, then the hook-review prompt reads as pending, then any composer line reads as ready.
  - `TrustDismissSequence`: zero steps gives Enter alone, a negative distance gives Up, a missing caret or option gives nothing, and a distance over 1 gives nothing.
  - `OwnPastePlaceholder`: compares the rune count of the text.
  - `PaneTooShort`, `IdleSession` and `RunningTurn` behave as documented.
  - Correct against the code. Whether the `InputBoxText` continuation-line rule matches the real status line under the composer rests on the spike captures, which I did not read.
- `registry.builtins`, `ConfigSpec`, `knownEngines`, `shuttleengines` registry and scrub (modelspec and shuttleengines changes): the Codex defaults map is fresh per alias, `CategoryNone` classifies the three new aliases, and the scrub is `CODEX_` as a prefix. Correct.
- `fslink` `CreateFileLink` (`fslink_linux.go`, `fslink_windows.go`): Linux uses the same guards as `CreateDirLink` and writes through. Windows refuses before touching anything and names the way forward. Correct.
- `StartupChecker` seam (`run.go:746`): the call is a type assertion on the engine, so existing engines are unaffected. Correct. The unawaited start path (`TestRunner_StartSession`) may not go through `launch`, and I did not read it.

## Findings

1. **LOW, design.** `TurnStartInterrupt` can read a stale abort as the current turn's (`rollout.go:98-116`).
   - When either id is missing, or `turnStart.At` is zero, the fallback has no upper bound and no lower bound.
   - It then takes the newest `task_complete` or `turn_aborted` in the whole rollout.
   - A still-running turn after an earlier aborted one therefore reads as interrupted.
   - With turn ids on both sides the match is exact and correct.
   - Evidence: `rollout.go:104-110`.
2. **LOW, design.** Hook stamps can be lost when two hooks interleave (`events.go:79-87`, `93-99`).
   - Any payload line resets `holdFrom` to -1 while another hook's stamp is still unmatched.
   - `consumed` then moves past that stamp, and when its payload arrives later it reads a zero time.
   - Sequential hooks, which are the normal case, are unaffected.
   - Evidence: `events.go:86`.

Beyond these, every other branch ran as intended.

## Tool calls

1. Read `internal/shuttleengine/run.go:690-780`: my context lacked where `CheckAfterStartup` sits in `launch`.
2. Grep `internal/shell/*.go` for `Chain`, `Exit` and `WithEnv`: my context lacked what the launch line's chaining and env prefix produce.
