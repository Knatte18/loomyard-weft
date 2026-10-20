# Card 2 — darn-tree-scope-config

**Create:**
- `internal/darnengine#ErrTreeScopeWithoutTrees`
- `internal/darnengine#ErrTreeScopeKey`
- `internal/darnengine#TreeScopedCommand`
- `internal/darnengine#WithEditorWayForward`
- `internal/darnengine#TestTreeScopedCommand`

**Edit:**
- `internal/darnengine#Config`
- `internal/darnengine#ConfigOpenMaps`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#LoadConfig`
- `internal/darnengine#VerifyCommand`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine#TestVerifyCommand`
- `internal/configreg#TestLookup`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`

**Uses:**
- `internal/darnengine#seedDarnConfig`
- `internal/darnengine#templateConfig`
- `internal/darnengine#writeLocalDarnConfig`

**Intent:** Give `darn.yaml` the two tree-scope keys of D1 and the pure classification of D2.

`Config` gains `TreeScope map[string]string` (`yaml:"tree_scope"`), the worktree-relative tree paths and the test target each contributes, and `TreeScopeVerify string` (`yaml:"tree_scope_verify"`), the tree-scoped command holding `{targets}`.
The template ships both empty, `tree_scope: {}` and `tree_scope_verify: ""`, on lines after `verify_attempts`, so the first `verify: ""` in the template stays darn's own `verify` line, which `seedDarnHubConfig` and the darn tests replace by its first occurrence.
`tree_scope`'s comment says it maps a tree path to the target string the tree contributes, an empty target meaning the tree has no tests of its own, and that it is written with `lyx config darn --set tree_scope.<tree>=<target>` and removed only in `lyx config darn`'s editor; a tree path holding `.`, such as `.github`, is written in `lyx config darn`'s editor.
`tree_scope_verify`'s comment says it holds `{targets}`, that it must run with an empty `{targets}` since a docs-only or empty diff expands it so, that tree scope is on only when both keys are non-empty, and that `lyx config darn --set tree_scope_verify=` turns it off.
`ConfigOpenMaps` adds `tree_scope`, so `--set tree_scope.<tree>=<target>` writes one tree and reconcile carries the hub's trees whole; `TestLookup`'s darn row adds it to the wanted open maps.
`ConfigSpec` classifies `tree_scope_verify` as `configengine.CategoryExecutable`, as `verify` is, and `tree_scope` as `configengine.CategoryGuard`, so neither the local layer nor a task's `run_settings` sets either; its doc comment says so.

`LoadConfig` trims one trailing `/` from each tree key and returns `TreeScope` keyed by the trimmed paths, then refuses, in this order:
- a tree key that is empty, absolute, holds a backslash, or holds an empty, `.` or `..` segment after the trim, such as `tools//x`, `tools//`, `./tools` and `a/../b`: the error wraps `ErrTreeScopeKey` and names the key as written, quoted, with no way forward;
- two tree keys equal after the trim, such as `tools` and `tools/`: the error wraps `ErrTreeScopeKey` and names both keys as written, in sorted order, with no way forward;
- a non-empty `tree_scope_verify` while `tree_scope` holds no tree: the error wraps `ErrTreeScopeWithoutTrees` and names `lyx config darn --set tree_scope.<tree>=<target>` in the same call as the command, or `lyx config darn --set tree_scope_verify=` to turn tree scope off, and says a tree path holding `.`, such as `.github`, is written in `lyx config darn`'s editor;
- a non-empty `tree_scope_verify` without `{targets}`: the error names `lyx config darn --set tree_scope_verify=<command holding {targets}>`.
Both keys empty, and trees with an empty `tree_scope_verify`, load with tree scope off.
`WithEditorWayForward(err)` returns an error wrapping `ErrTreeScopeKey` with the editor way forward appended, "fix the named tree_scope key in `lyx config darn`'s editor from the prime, then re-run the lyx command that refused", since `--set` cannot remove a key the hub file holds, and returns every other error, nil included, unchanged; every reader of the hub `darn.yaml` that refuses on a load error calls it, directly or through `VerifyCommand` (this card's `VerifyCommand`, and Card 7's `wireDarn`, `loomengine.RecipeOverride` and `perchHardCap`), except a reader's second load of the same files in one call, which only an edit between the two loads makes fail.
`VerifyCommand` returns `WithEditorWayForward`'s error for a load error wrapping `ErrTreeScopeKey` instead of appending `verifyWayForward`; every other load error keeps `verifyWayForward`.

The existing descriptions of `verify` become false once the gates run a scoped command (Card 7) and Merge-In and Finalize resolve per diff (Card 8), so this card restates them: `Config.Verify`'s field comment, `verify`'s template comment, `config.go`'s header lines on `VerifyCommand`, and `doc.go`'s sentences on `verify` and `VerifyCommand` say `verify` is the hub's whole verify command, which a darn gate runs where `impactset` falls back and Merge-In and Finalize run for a diff that is not tree-scoped, and which stays the gates' base command.

`TreeScopedCommand(cfg, paths)` is told the loaded config and the task diff's changed paths and returns the tree-scoped command and whether the diff is tree-scoped.
It is told a config `LoadConfig` returned, so its tree keys carry no trailing `/` and a non-empty `TreeScopeVerify` comes with at least one tree.
Tree scope is off, and the answer `("", false)`, when `TreeScopeVerify` is empty.
A path lies in a tree when it equals the tree path or starts with the tree path and a `/`.
The diff is tree-scoped when every path lies in some tree, and an empty path list is tree-scoped.
The command is `TreeScopeVerify` with every `{targets}` replaced by the non-empty targets of the touched trees, deduplicated, in tree-path order, joined by single spaces, so one diff always yields one string; no path text reaches the command.

`doc.go` describes the two keys, their categories, the refusals and which caller adds which way forward, the classification, and the two operator commands: `lyx config darn --set tree_scope.tools=./tools/... --set tree_scope.docs= --set "tree_scope_verify=lyx gate test --tags integration --guards {targets}"` turns tree scope on for this hub, and `lyx config darn --set tree_scope_verify=` turns it off.
It also states D2's bound: a tree-scoped diff gets no whole-module run before it lands, so a break outside the listed trees that neither the trees' tests nor the guard-marked scans exercise can reach the parent, bounded by what the hub lists as non-production.

`TestLoadConfig` gains rows over the hub file: the template loads `TreeScope` empty and `TreeScopeVerify` empty; trees with an empty `tree_scope_verify` load; a key written `tools/` loads as `tools`; a non-empty `tree_scope_verify` with no tree is refused naming both commands and the clause that a tree path holding `.` is written in `lyx config darn`'s editor, and wrapping `ErrTreeScopeWithoutTrees`; a non-empty template without `{targets}` is refused naming the `--set tree_scope_verify=<command holding {targets}>` command; each of `""`, `/abs`, `tools//x`, `tools//`, `./tools`, `a/../b` and `tools\x` is refused naming the key as written and wrapping `ErrTreeScopeKey`; `tools` beside `tools/` is refused naming both.
Its local-layer subtest's key map gains `tree_scope` and `tree_scope_verify`, each refused naming the local file and the key.
`TestVerifyCommand` gains a row: a hub file with a malformed tree key returns an error naming `lyx config darn`'s editor and the clause "then re-run the lyx command that refused", and not `--set verify=`; that row pins `WithEditorWayForward`'s wrapping branch, and the existing rows its pass-through.
`TestTreeScopedCommand` is a table: tree scope off with an empty `tree_scope_verify`; `docs` and `tools` trees with a docs and a tools path yield `lyx gate test --tags integration --guards ./tools/...`; two trees sharing one target yield it once; targets come in tree-path order whatever the path order; `tools` does not claim `toolsx/a.go`; a path equal to a tree path lies in it; `docs/ä.md` lies in `docs`; one production path makes the diff not tree-scoped; an empty path list yields the command with an empty `{targets}`.
`TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains a darn row, so the reconcile `lyx config reconcile` runs, `MigrateConfig` included, is the code under test: a present hub `darn.yaml` holding `tree_scope` with `tools: ./tools/...` and `docs: ""` reconciles with seed `SeedHub` and keeps both trees and their targets.

**ImpactSummary:** darnengine gains two keys, two refusal sentinels, a way-forward wrapper and a pure classifier; its loader refuses only values no file holds today, configreg's open-map pin moves with the new open map, and configsync gains one darn reconcile row.
