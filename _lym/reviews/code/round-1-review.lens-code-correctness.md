```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed branch on a concrete input. I found no defects.

**Function / branch, case run, verdict**
- `upgradeStore`, `["claude-file"]` with no version: becomes `["agents-file"]`. Correct.
- `upgradeStore`, `["codex-file","claude-file"]`: codex dropped, claude read as agents, result `["agents-file"]`. Correct.
- `upgradeStore`, `["claude-prompt","claude-file"]`: order kept. Correct.
- `upgradeStore`, `["codex-file"]` alone: unchanged, since the `Contains` guard fails. Correct.
- `upgradeStore`, nil `Targets` or an empty store: nothing changes, the version is set. Correct.
- `upgradeStore`, an already versioned store: returned as is. Correct.
- `Decode`, `Load` and the replayer's four reads (`replayer.go:71-83`): all go through `upgradeStore`. Correct.
- `Store.MarshalJSON`, versions 0, 1 and a higher future version: `max(Version, storeVersion)` writes the version and keeps a higher one. Correct.
- `Render` loop: `agents-file` has `RendersInto`, so it is skipped and has no `File` to dereference. Correct.
- `Render` with `Only=["agents-file"]`: renders nothing. No caller passes it; the seam only passes names it selected from `ReRender`. Not a defect.
- `Render`, a claude-file-only entry: `AGENTS.override.md` is not written, and a marked stale copy is removed. Correct.
- `fileText`, an entry aimed at both `claude-file` and `agents-file`: `reachesFileTarget` returns once per entry, so no duplicate text. Correct.
- `fileText`, `claude-file`: the `@AGENTS.md` import is skipped when `CLAUDE.md` or `.claude/CLAUDE.md` exists. Correct.
- `fileText`, `codex-file`: reads `AGENTS.md`, else `CLAUDE.md`, and does not read `.claude/CLAUDE.md`. Consistent with the table's `ReRender` for `codex-file`.
- `Seam.TookIn`, `.claude/CLAUDE.md`: selects `claude-file` only. Correct; the shared target has no `ReRender`, so it is never selected.
- `Show("agents-file")`: refused with a message naming `claude-file, codex-file` and the way forward. Correct. Show for a prompt target and for a JSON target are unchanged.
- `TargetFiles`, `agents-file`: concatenates the files of the two targets it renders into. A prompt or unknown name gives nil. Correct.
- `targetReach`, `claude-file`: "also carries `agents-file`". For `agents-file` itself it lists both files. Correct.
- `fileTargets` and `fileTargetHelp`: only targets with files are listed, and `claude-settings` is included. Correct.
- `ReadLayers`: shipped layer entries are not passed through `upgradeStore`, so shipped `claude-file` entries stay `CLAUDE.local.md`-only, as before. Correct.

**Findings:** none.

**Tool calls (2)**
1. Grep for `Only`, `.File`, `LookupTarget` and `Delivery` readers, plus `overview.go` lines 60-120. The manifest lacked other readers of `Target.File` and the overview loop.
2. `seam.go` lines 30-110 and `ReadLayers`. The manifest lacked the seam's target selection and whether the shipped layer is migrated.
