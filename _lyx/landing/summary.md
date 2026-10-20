# Darn verify gates scope to the diff: impacted set and tree scope

A darn run no longer pays for the whole-module suite at every verify gate.
The writer and fix-round gates run the diff's impacted set plus the repo-wide scans, and a diff that touches no production code runs only its trees' tests plus the scans at every site, landing included.
Before this change darn's gates ran the hub's whole `verify` at Darn-Write and at each Darn-Burler-Fix round, and again at Merge-In and Finalize, whatever the diff touched.

## What changes

- **Tree scope, off by default.**
  `darn.yaml` gains two hub-wide keys.
  `tree_scope` maps a worktree-relative tree path to the test target the tree contributes, such as `tools: ./tools/...` and `docs: ""`.
  `tree_scope_verify` is the scoped shell command holding `{targets}`, which must run with an empty `{targets}`.
  Tree scope is on only when both are non-empty, and both are closed to the local layer.
- **Classification.**
  `internal/darnengine` classifies the task diff against the trees as a pure function.
  A diff is tree-scoped when every changed path lies in some tree, and an empty diff is tree-scoped with no targets.
  `{targets}` is filled only from config values, deduplicated and in tree-path order, so a diff path never reaches the shell and the same diff always yields the same string.
  A new `internal/fabricengine` read lists both sides of a rename, verbatim, so a rename or deletion out of production code counts as production.
- **Writer and fix-round gates.**
  Darn-Write and Darn-Burler-Fix run the tree-scoped command for a tree-scoped diff.
  Otherwise they run the command `impactset.Derive` gives from the merge base: touched packages, their reverse dependencies and every `//lyx:guard` test.
  Where `impactset` names a fallback, the gate runs the hub's whole `verify`.
  The told `verify` stays each gate's base command, so a pass keeps its verified-tree record.
- **Derived command runs through `lyx gate test`.**
  `impactset`'s test and guard steps now go through `lyx gate test --tags integration`, inheriting the held slot, the prebuilt `lyx` and the warm test cache.
  This also changes the command loom's round-form gate runs at Webster-Burler-Fix and Merge-Burler-Fix; nothing else of loom changes.
- **Landing sites.**
  Merge-In and Finalize resolve their command per call: the tree-scoped command for a tree-scoped diff, else the whole `verify`.
  Publish classifies the diff after its moved-parent check and skips `publish_verify` for a tree-scoped diff, removing the Publish failure record.
  Loom's Publish runs it as before.
- **`lyx gate test --guards`.**
  After the named packages it runs every top-level `//lyx:guard` test in its own package, inside the same slot.
  The package list may then be empty, and the verb refuses `--guards` without `integration` among its tags.
  A guard-scan failure refuses the verb before any slot, naming the file and the way forward.
- **Hub-wide editor save is load-checked.**
  `lyx config <module>`'s editor save now loads the written file through the module's loader and rolls back a write that leaves it failing with a new error, as `--set` already did.
  The refusal keeps the staging copy and names its path.
  This applies to every hub-wide module.
- The darn writer stencil, `PATTERN-test-speed`, `PATTERN-verified-tree`, `docs/overview.md` and the module docs are restated to match.

## Design decisions for review

- Tree, never import graph, decides the last gate: a diff touching production code still meets the whole `verify` at Merge-In, and the impact-scoped pass only replaces the writer and fix-round gates.
- **Bound:** a tree-scoped diff gets no whole-module run before it lands.
  A break outside the listed trees that neither the trees' tests nor the guard-marked scans exercise can reach `main`.
  This ends `PATTERN-test-speed`'s "caught by the full plan verify at Publish or Finalize" for such diffs; the hub bounds it by what it lists as non-production.
- A tree-scoped run lands with no `tmux`-tier run, and its Merge-In does not whole-verify parent code merged into the task.
- `--guards` scans for the `//lyx:guard` marker rather than taking a hand-kept package list, which would go stale as scanning tests are added.
- An unmarked scanning test is not run by the impact-scoped gate, as for loom's round gate today.
- Non-Go hubs get tree scope from config, and their writer gates keep the whole command because `impactset` falls back without `go.mod`.
- The gain is unmeasured: the build and vet passes of the impact-scoped command still run at each gate.
- `darn-verify-parent-fix` and `darn-verify-tier-unrunnable` are not settled here.

## Turning tree scope on or off for this hub

This hub's `darn.yaml` values are an operator step from the prime after landing.
On:

```
lyx config darn --set tree_scope.tools=./tools/... --set tree_scope.docs= --set "tree_scope_verify=lyx gate test --tags integration --guards {targets}"
```

Off, leaving the trees inert:

```
lyx config darn --set tree_scope_verify=
```

`--set` never removes a key from an open map, so removing a tree needs `lyx config darn`'s editor.

## To check by hand

- Run `lyx gate test --tags integration ./...` and the tree-scoped command on a real tools-only darn diff; neither was run against a live darn run.
- Measure the gain on OSL-1046, which the board leaves to the operator.
- `treescopedcommand-nested-trees-drop-target`, in the `internal/darnengine` tree-scoped command expansion: check that nested tree keys, such as `tools` and `tools/x`, each contribute their target.
- `wiredarn-verify-gate-told-scoped-command`, in `internal/loomcli/darnwiring.go`: check that the darn verify gate is told the hub's whole `verify`, not the scoped command.
- `darn-tree-scope-drops-verify-way-forward`, in the darn tree-scope load refusals: check that each names its way forward from the prime.
- `landing-sites-editor-way-forward-untested`, at the landing sites in `internal/loomcli`: check that the editor refusal's way forward is reached by a test.
