# Card 1 — vscode-attach-chain

**Create:**
- `plan:internal/vscode#TaskChain` -> `type TaskChain int`
- `plan:internal/vscode#TaskChainInteractive` -> `const TaskChainInteractive TaskChain = 0`
- `plan:internal/vscode#TaskChainAttachOnly` -> `const TaskChainAttachOnly TaskChain = 1`

**Edit:**
- `internal/vscode#WriteConfig`
- `internal/ideengine#Spawn`
- `internal/vscode/config_test.go`

**Intent:**
Give `vscode.WriteConfig` a chain selector, the `TaskChain` type in `internal/vscode/config.go`, with exactly two values:
`TaskChainInteractive` (the zero value, today's chain) and `TaskChainAttachOnly`.
Declare them as one iota const block, `TaskChainInteractive TaskChain = iota` then `TaskChainAttachOnly`, so `TaskChainInteractive` equals 0 and `TaskChainAttachOnly` equals 1, as the handles' heads say.
`WriteConfig` gains a trailing `chain TaskChain` parameter.

`TaskChainInteractive` keeps today's behaviour byte-for-byte: settings and tasks each written only when absent, the four-task `reed up` → `reed add claude` → `reed attach` chain, and the `gitignore.Ensure(dir, ".vscode/")` call.

`TaskChainAttachOnly` is what batten's driven pair needs:
- `settings.json` keeps the write-only-when-absent rule, identical content.
- `tasks.json` is written unconditionally, overwriting any existing file, and holds exactly one task:
  label `reed attach`, `type: shell`, `command` the resolved `lyxPath` (with the same empty-string → `"lyx"` fallback), `args: ["reed", "attach"]`, the same `presentation` block today's `reed attach` task carries, and `runOptions: {"runOn": "folderOpen"}`.
  No `reed up`, no `reed add` and no `dependsOn` task.
  Rationale: `reed attach` boots a cold session itself, and `reed add claude` would put a second, bare claude strand beside the child's driver.
- No `gitignore.Ensure` call: the driven pair's `.vscode/` is kept out of git through `info/exclude` by the caller (card 4), never by editing a tracked `.gitignore`.
- `claudePath` is unused; the caller may pass `""`.

Whether an existing `tasks.json` is tracked is not this package's question — it never runs git; the caller decides whether to call `WriteConfig` at all.
Update `WriteConfig`'s doc comment to describe both chains and the overwrite rule, and the one production caller, `ideengine.Spawn`, to pass `vscode.TaskChainInteractive` (plain `lyx ide spawn` output unchanged).

Tests in `config_test.go` (untagged, `t.TempDir()` only, no git):
update every existing `WriteConfig` call to pass `TaskChainInteractive`, with no assertion changed, which proves the interactive chain is unchanged;
add attach-only tests asserting the decoded `tasks.json` holds exactly one task with `command` equal to the given `lyxPath`, `args` exactly `["reed","attach"]`, `runOptions.runOn` exactly `folderOpen`, and no task labelled `reed up` or `reed add claude`;
that an empty `lyxPath` falls back to `lyx`;
that a pre-existing interactive `tasks.json` is overwritten with the attach-only one while a pre-existing `settings.json` stays byte-for-byte identical;
and that no `.gitignore` is created or modified.

**ImpactSummary:** `WriteConfig` gains one parameter; its only production caller is `ideengine.Spawn`, updated here to keep today's chain, plus the package's own tests.
