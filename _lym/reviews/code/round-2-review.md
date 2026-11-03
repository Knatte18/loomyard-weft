---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/codexengine.go:117
    summary: the told records checkout goes onto the launch line unresolved, while the trust roots are symlink-resolved because Codex keys a linked path by its resolved target
    fix: In Prepare, when c.dirs.Records is non-empty, resolve it with filepath.EvalSymlinks and use the resolved path as the writable root, falling back to the told path and logging a Warn when the resolve fails (a records checkout that cannot be resolved is not worth refusing the launch for). Add one row to TestPrepare_Launch, in internal/shuttleengine/codexengine/prepare_test.go, whose told Records is a symlink to the real directory and whose want is the resolved path.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/junctionnames.go:125
    summary: ConfigDirs doc says a stat failure other than absence is logged, but only the Bolt stat logs; a failing weft stat silently leaves Records empty, so a Codex launch drops its writable root with no trace
    fix: In ConfigDirs, treat the weft stat the way the Bolt stat is treated (set Records on a directory, log a logger.Warn naming the path and error when the error is not os.IsNotExist, stay silent on absence), or narrow the doc comment's third sentence to say it covers the Bolt checkout alone. The first is preferred, since an unlogged empty Records turns into a missing report with nothing in the log to explain it.
---

### [LOW] The records writable root is not symlink-resolved

**Location:** internal/shuttleengine/codexengine/codexengine.go:117

**Issue:** `readCheckout` resolves the worktree and main checkout roots with `EvalSymlinks` "as Codex resolves them", because Codex keys trust on the resolved path (decision record, Constraints: "Codex keys a linked path by its resolved target").
The writable root is the one path of the launch that is not resolved: `Prepare` passes `c.dirs.Records` as `fabricengine.ConfigDirs` built it, `weftname.SiblingPath(hub, base)`.
A hub reached through a symlinked parent, such as a symlinked home or code directory, then hands Codex a path whose resolved form differs from the one in the `-c` setting.
The decision record marks the sandbox's treatment of this root as proven only by the build's `codex sandbox` run, and nothing in the diff or the tests shows that run used an unresolved path.
The tests cannot see the gap either: the fixture resolves its temp dir first (`newRepoFixture`), so every `Records` they pass is already resolved.
The code does meet `PATTERN-cwd-resolution`: the engine imports no `lymcwd`, reads no link, and is handed the absolute path as told geometry through `Dirs.Records`.
This is a robustness gap against the decision's weft-checkout writable root, not a violation of it.

### [LOW] ConfigDirs documents logging it does not do for the weft checkout

**Location:** internal/fabricengine/junctionnames.go:125

**Issue:** The doc comment's last sentence reads as covering every stat `ConfigDirs` makes: "A stat failure other than absence is treated as absence and logged."
After this diff `ConfigDirs` makes two stats, and only the Bolt one logs (`logger.Warn` in the `switch`).
The weft stat in `ConfigDirs` drops any error, so a permission or I/O error leaves `Records` empty, the Codex launch line carries no `sandbox_workspace_write.writable_roots`, and the assistant's later report write fails inside the sandbox with nothing in the log that points back to the missing root.
`TestConfigDirs` pins presence and absence of the weft checkout but not the failure path, so the doc comment and the code disagree unchecked.

## Focus departures

None.
Settled on the two focus questions:

- The engine is handed an absolute path as told geometry (`Dirs.Records`, filled by `fabricengine.ConfigDirs`) and reads neither `.lym` nor `_lym`; no `fslink.IsLink` or `fslink.PointsTo` call is left in `home.go` or `codexengine.go`.
  `PATTERN-cwd-resolution` holds (no `lymcwd` import in `codexengine`; the path comes from the hub's wiring), and the decision's weft-checkout writable root is met, with the empty case outside a hub adding no root.
  The unresolved path is F1 and the silent stat failure is F2.
- `prepare_test.go` pins the launch shapes outside a linked worktree to the extent the engine now varies by them: `TestPrepare_Launch` has `RecordsNotTold` for the no-hub shape, `TestPrepare_ReadiesHome` has `PlainClone` for a repository with no main checkout, and `TestPrepare_RefusesBeforeWriting` has `CommittedRulesUnderSubfolderAnchor` for a `.codex/rules` directory under an anchor that differs from the worktree root.
  No test builds real `.lym` and `_lym` directories, and none is needed: `Prepare` no longer reads them, so the launch line cannot depend on them.
  Not a defect.
