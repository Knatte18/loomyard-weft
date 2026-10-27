```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, I looked for what calls, reads or asserts it outside the touched lines.

**Per-change affected set**

- **`composeNotice` renamed to exported `MessageLine`:** no reference to the old name is left anywhere. Callers are `typeNotice` and `hookread.go`. Covered.
- **`buildSettings` gains `creelFlagDirPosix`:** the one production caller (`claudeengine.go:208`) passes it. All test callers pass the new argument, including the dedicated flag-dir case at `settings_test.go:228`. Covered.
- **`PostToolUse` key in `settingsHooks`:** the field has `omitempty`, so a standalone run emits no key. `hookEventPostToolUse` is added in `signals.go`. Other tests that assert the full hooks shape pass `""` and still hold. Covered.
- **`ToolBoundaryContextRenderer` capability:**
  - Declared in `engine.go`.
  - Satisfied by `Claude` (`capabilities.go` assertion).
  - Reached through `shuttleengines.ToolBoundaryRenderer`.
  - Listed in `shuttleengine/doc.go` and `shuttleengines/doc.go`.
  - The only consumer is `hookread.go`.
  - Covered.
- **`Geometry.FlagDir` added:**
  - Constructors: `hubgeom.CreelGeometry` sets it. Test literals that omit it get no flags, because `syncFlagsLocked` returns early on an empty `FlagDir`.
  - `hubgeom_test.go` asserts the new field, and `lymdirs.CreelFlagDirName` backs it.
  - `ReconcileFlags` does call `MkdirAll` on an empty `FlagDir`. Only the daemon's `Run` calls it, and `Run` always gets the hub geometry, so this cannot trigger. Covered.
- **`journalWriter.appended` and the `Store.do` sync:** `appended` is only read in `do`. Every journaled change goes through `do` (`Ack`, `Release`, `markNoticeRead`, the post, forward, withdraw and return verbs). Covered.
- **`markNoticeRead` no longer refuses on `reserved[id] != ""`:**
  - The only caller is `typeNotice`, which now reserves first.
  - `paneReserver` is `"\x00pane"`, so it is never an agent name.
  - Every other reader of `reserved` (`unreadLocked` at `pool.go:400`, the notice scan at `deliver.go:432`, and the new reserve, release and mark helpers) treats a non-empty value as reserved. `Read` and the claim both go through `unreadLocked`.
  - Covered.
- **`runVerb` refactored into `hubClient`:** it keeps the same hub refusal text and order. `hookReadCmd` reuses it. Covered.
- **New `VerbClaim`, and `read` split into `handOut`:** the dispatch case is added, and `handOut` is shared by `read` and `claim`. Covered.
- **New hidden `hook-read` verb:** it is listed in the `cli.go` group help, `creelcli/doc.go` and `docs/overview.md`. I did not check the registration test for hidden verbs. Covered.
- **Docs and `PATTERN`:** `creelengine/doc.go` and `claudeengine/doc.go` describe the flag, the claim and the hook. The diff touches no file under `pattern/`, `PATTERN.md`, `contracts/` or `tools/`.
  - `pattern/PATTERN-pane-text-injection.md` makes no claim about notice reservation, so it needs no edit.
  - `tools/sandbox/SANDBOX-CREEL-SUITE.md` asserts the orch's inbox counts. Reserved messages already left the unread count before this diff, so it is unaffected.
  - The orch stencils that name `lym creel read` stay correct, since the idle path is unchanged.
  - Covered.

**Findings**

None. I found no member of the affected set that the diff leaves wrong.

**Tool calls (3 of 6)**
1. Ran `grep` for `composeNotice`, `buildSettings(`, `markNoticeRead` and its siblings, `PostToolUse` and `hook-read` across the repo. My context lacked callers and test sites outside the diff.
2. Ran `grep` for `Geometry{` literals, `VerbRelease` consumers, the `creelengine/doc.go` flag text and stencils that name `lym creel`. My context lacked the geometry constructors and the docs set.
3. Ran `grep` on `pattern/PATTERN-pane-text-injection.md` and `SANDBOX-CREEL-SUITE.md`, and a `git diff --stat` over `pattern`, `PATTERN.md`, `tools` and `contracts`. My context lacked their contents and whether the diff touches them.
