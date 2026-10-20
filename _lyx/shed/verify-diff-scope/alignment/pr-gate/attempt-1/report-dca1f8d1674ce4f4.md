---
verdict: APPROVED
---

The diff of pull request 774 at `b288ee6864c89e8736dfb452135b02f38bea1a30` against `main` builds the board entry's goal and the decisions D1 to D6, and nothing the goal does not call for.

- D1: `internal/darnengine/config.go` adds `tree_scope` and `tree_scope_verify`, closed to the local layer, off by default, with the `LoadConfig` refusals the record lists; `tree_scope` is registered as an open map (`internal/configreg/configreg_test.go`).
  `internal/configcli/hubwide.go` adds the `CheckLoad` check around the editor save, its own sentinel `errHubFileLoad`, the kept staging copy and the editor way forward.
- D2: `fabricengine.ChangedPathsBothSides` reads with `-z --no-renames`; `darnengine.TreeScopedCommand` classifies at a path boundary and expands `{targets}` from config values only, deduplicated in tree-path order.
- D3: `internal/loomshed/verifygate.go`'s darn form runs the lints from the merge base, then the tree-scoped command with no `tmux` pass, else `impactset.Derive`'s command, else the told `verify`; `wireDarn` keeps the told command as the hub's whole `verify`, so it stays the base command.
  `internal/impactset/derive.go` moves the test and guard steps onto `lyx gate test --tags integration`; loom's round form changes only by that derived command, since a loom run leaves `VerifyTreeScope` nil.
- D4: `darnVerifySource` resolves the Merge-In and Finalize command at each call, and `PublishVerifySkip` lets Publish take the unset-`publish_verify` path for a diff tree-scoped at that call; loom's source leaves it nil.
- D5: `lyx gate test --guards` in `internal/gatecli` scans with `impactset`'s guard scan, refuses without `integration` and refuses a scan failure before any slot.
- D6: the darn writer stencil, `pattern/PATTERN-test-speed.md`, `pattern/PATTERN-verified-tree.md` and `internal/configcli/doc.go` are restated.

The Out list holds: loom's recipe is untouched apart from the derived command, no non-Go test selection is added, and this hub's `darn.yaml` values are left to the operator, with the on and off commands in the change description.

The carried-over open risks read as follows against the diff.
The Webster-Review items are addressed in code: nested trees each contribute their target, and the darn gate's told command is the whole `verify`, with the scoped command reaching the gate through `VerifyTreeScope`.
The Discussion-Review and Plan-Review items concern the editor refusal's way forward and the verify-command way forward, which the diff carries in `hubFileLoadRefusal` and in `darnTreeScope`'s leading `VerifyCommand` read; neither is a scope deviation.
The unmeasured gain and the unsettled `darn-verify-parent-fix` and `darn-verify-tier-unrunnable` are left open by the record itself.

## Overlap

`lyx loom peers` lists no peer run in flight and no landed parent commit, at `own_row` 21, so there is no overlap and no landing order to give.
The board entries `lenses-module`, `lyxbin-timeout-test-xdr` and `watchdog-reap-test-xdr` depend on this task; that is an ordering, not an overlap.
`docs/overview.md` changes as a shared bookkeeping file only.

No board entry is made obsolete: `darn-verify-parent-fix`, `darn-verify-tier-unrunnable`, `dotnet-scoped-verify` and `gate-language-neutral` stay open by the record's Scope and Open risks.
