```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff's changed functions and branches for correctness (code-correctness lens).

**Cases run, function or branch by branch**

- `openSteersWithWholeCommandRefusal` (`internal/shellguard/rows.go:386-398`):
  - **Empty rows slice:** the loop does not run and the empty slice comes back. Correct.
  - **Row with nil `EngineTexts`** (raw-go, computed-rm, machine-sleep, adopt-take-hold, publish, refuse-raw): ranging over a nil map is a no-op, and only the plain `Steer` gets the prefix. Correct.
  - **Row with a Codex `EngineText`** (python, shell-edit, headless-claude): the codex value is copied, its `Steer` prefixed, and the copy written back under the key it was ranged from. Go allows that write during a range. `SteerFor("codex")` returns the prefixed engine steer. Correct.
  - **`EngineText` with an empty `Steer`** (no shipped row has one): the `text.Steer != ""` branch skips it, and `SteerFor` falls back to the already-prefixed plain steer. No double prefix and no bare clause. Correct.
  - **`Rows` called twice:** each call builds fresh map literals and row values in `rowsBeforeRefuseRaw`, `rowsAfterRawGo` and `refuseRawRow` (`rows.go:474-580`). The prefix never stacks on a map left over from an earlier call. Correct.
  - **Notices:** the function never touches `Notice` or `EngineText.Notice`. Each notice is built before the prefix is applied:
    - raw-go: `rawGoSteer + "."`, at `rows.go:494`;
    - shell-edit on Codex: `codexSteerShellEditDeny + "."`;
    - headless-claude on Codex: `codexSteerHeadlessClaudeDeny + "."`;
    - publish: `steerPublishDeny() + "."`.

    None carries the clause. Correct.
- **`Rows` with an empty or nil `refuseRaw`** (`rows.go:413-420`): no refuse-raw row is built, and the rest are prefixed. **With a non-empty list**, the refuse-raw row sits in the slice that is passed in, so it is prefixed too. Correct.
- **`checkTexts` and `init` on the new text** (`rows.go:583-608`): `wholeCommandRefusal` holds no `'`, `"` or `\`, so every prefixed steer still passes the forbidden-character check. Since it only touches steers, the notice newline rule is unaffected. `init` cannot panic because of this change. Correct.
- **Claude renderer** (`claudeengine/settings.go:357-377`): `denyJSON(row.Steer)` sits inside `echo '…'`. The clause adds only `:` and `.`, which are safe in that single-quoted shell string and in JSON. The golden files are byte-identical once the clause is stripped, as the handler's `cmp` check showed. Correct.
- **Codex renderer** (`codexengine/guards.go:57-71`): `posix.Quote(denyDecision(row.SteerFor("codex")))` holds no quote or backslash from the clause. Correct.
- **Test change, `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks`:** `row.Notice != strings.TrimPrefix(row.Steer, wholeCommandRefusal)+"."` reproduces the pre-prefix relation of `Notice` to `rawGoSteer`. Correct.

**Findings**

None.

**Tool calls**

None. The manifest context held every definition and caller this lens needed.
