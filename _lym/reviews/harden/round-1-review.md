---
verdict: APPROVED
---

## Probe log

Probes are recorded as they are made.

1. Read the charter, the focus, `PATTERN-single-repo-hub`, `PATTERN-fabric-boundary`, `PATTERN-code-conventions`, `internal/fabricengine/clone.go`, `internal/fabricengine/recordslayout.go` and `internal/hardencli/cli.go`.
   `lym harden sandbox new --help` says it builds a hub "from two local bare repositories".
   `sandboxDeps` clones through `fabriccli.CloneAndWireFromLocalBares(container, codeBare, recordsBare, ...)`, the two-repository form.
2. `lym harden sandbox new` (13 s, rebuilt `.dev-bin/lym`).
   Observed: hub `.lym/harden-sandbox/hub-1423524032/warp-bare-LYMHUB` over `warp-bare` and `weft-bare`, a two-repository hub.
   The recipe has no flag or verb that makes a one-repository hub.
3. To drive the one-repository seed by hand: `git clone --bare` of the sandbox's `warp-bare` into `.lym/harden-sandbox/one-a/code.git`,
   then `.dev-bin/lym fabric clone --single-repo --shortname sbo .../one-a/code.git` from `one-a`.
   Observed: `{"error":"lym fabric clone is refused from the task session ly:harden-fabricengine:harden-burler-review; build a scratch hub as a hubforge test fixture instead, ...","ok":false}`.
   `internal/fabriccli/roleguard.go#sandboxedRoleRefusal` refuses every `lym fabric` verb outside `FabricReaderVerbs` (list, pairs, status, diff, shortname, prune and cleanup without `--apply`) from any slugged non-driver, non-orch strand.
   So the review half of a hardener run cannot drive clone, add, push, checkout, remove or cleanup `--apply` against a sandbox hub, and I did not work round the guard.
   `rm -rf` of `one-a` was denied by the permission layer, so `.lym/harden-sandbox/one-a/code.git` is left in the sandbox root for `lym harden sandbox clear` to remove; it is outside the root's hub list.
