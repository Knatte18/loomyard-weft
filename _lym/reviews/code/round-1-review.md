---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configcli/configcli.go:560
    summary: "No test pins that lym config reconcile sets PruneUnknown, in the hub worktree-file path (configcli.go:560) or the default-layer path (hubwide.go:184); removing the flag from both fails no test"
    fix: "Add a row to the existing default-layer reconcile integration test in internal/configcli/configcli_integration_test.go that seeds a board file holding a key the template lacks and no retired key covers (for example newer_key: 1), runs reconcile --layer default --apply and asserts the key is gone. Add one row for a hub worktree file in the hub-mode reconcile test, or a new fixture if none exists, asserting the same removal. The existing reconcile_integration_test.go:63 case is standalone and does not reach these call sites."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubreconcile/walk.go:285
    summary: "No test pins that the automatic callers (walk worktree reconcile at walk.go:285, clone.go:58 and :122, fabric.go:748) leave PruneUnknown unset for worktree files; the walk integration test checks only the board fabric.yaml"
    fix: "In internal/hubreconcile/ensure_integration_test.go extend runStaleBuildReconcile so a pair worktree's sparse config file also holds a key the template lacks, and assert it survives hubreconcile.Ensure. Add one row to the fabriccli reconcile test so the fabric.go:748 call site is covered the same way. A clone row is optional."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/configsync/configsync.go:641
    summary: "pruneSparse returns the DropRetired bytes unpruned when something is preserved, so a retired key that empties a mapping leaves an empty mapping the plain Prune path would drop, and under an unknown parent that empty mapping is also reported as preserved; no test uses a nested key"
    fix: "In pruneSparse (configsync.go:636-654), when preserved is non-empty run yamlengine.Prune(template, withoutRetired, ...) once and return its pruned bytes instead of withoutRetired, taking preserved from that same call so the bytes and the list come from one pass and an emptied mapping is dropped. Add a row to TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked or TestReconcileAll_SparseLayers with a nested retired key beside a nested unknown key, asserting Removed and Preserved are disjoint and the file bytes carry no empty mapping."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-config-layers.md:50
    summary: "The new rule says every other reconcile keeps an unknown key and reports it as preserved, but only the hubreconcile walk logs it, and only for a result that was applied; clone and lym fabric reconcile discard Preserved, and a run where every removal is preserved logs nothing"
    fix: "Reword the 'Keys the template lacks' section in pattern/PATTERN-config-layers.md and the hubreconcile/doc.go paragraph to say such a key is kept and carried in configsync's Result.Preserved, and that the walk logs it when it writes the file, instead of saying every reconcile reports it. Alternatively, make logResult's callers in hubreconcile/walk.go log a result with a non-empty Preserved even when it was not applied."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: PATTERN.md:21
    summary: "The PATTERN-config-layers entry says only lym config reconcile removes a base or worktree key the template lacks, but reconcileBaseWorktreeFile (standalone base, boardDir empty) still removes every such key through yamlengine.Reconcile and PruneUnknown does not gate it"
    fix: "Narrow the sentence in PATTERN.md line 21 and in pattern/PATTERN-config-layers.md 'Keys the template lacks' to a hub base or worktree file, and state in the ReconcileAll doc comment (configsync.go:405-419) that the standalone worktree base is fully reconciled and ignores PruneUnknown. No automatic caller reaches that path, since clone and the walk pass a board dir."
---

### [MEDIUM] No test pins that `lym config reconcile` sets `PruneUnknown`

**Location:** `internal/configcli/configcli.go:560`, `internal/configcli/hubwide.go:184`
**Issue:** The Goal's second half is that removal stays an explicit `lym config reconcile --layer default --apply`. The diff sets `PruneUnknown: true` at both operator call sites, but no test fails if either is dropped. The only unknown-key reconcile test is standalone and goes through the ungated `reconcileBaseWorktreeFile`; the default-layer integration tests seed retired or renamed keys, which are pruned without the flag.

### [LOW] Automatic callers leaving `PruneUnknown` unset are pinned for the board file only

**Location:** `internal/hubreconcile/walk.go:285`
**Issue:** `ensure_integration_test.go` asserts the board `fabric.yaml` keeps the newer key. Adding `PruneUnknown: true` to the walk's worktree reconcile, to clone or to `lym fabric reconcile` fails no test.

### [LOW] `pruneSparse` returns the retired-key-dropped bytes without a prune pass

**Location:** `internal/configsync/configsync.go:641`
**Issue:** When a retired key is the only child of a mapping, `DropRetired` leaves `a: {}`, which `yamlengine.Prune` removes on the plain path. The preserved branch keeps it, and under an unknown parent the empty mapping is itself listed as preserved. The effect is cosmetic and needs a nested retired key beside an unknown key, but the two branches return different bytes for the same removal.

### [LOW] The rule promises a preserved report that only the walk produces

**Location:** `pattern/PATTERN-config-layers.md:50`
**Issue:** The text says the clone and `lym fabric reconcile` report a kept key as preserved. They read only `Applied`, `Layer` and `Module` from each result, and the walk logs only applied results, so a preserved-only run is silent.

### [LOW] The rule's "base or worktree key" wording covers the ungated standalone base

**Location:** `PATTERN.md:21`
**Issue:** `reconcileBaseWorktreeFile` removes unknown keys from a standalone worktree base regardless of `PruneUnknown`. Only the manual `lym config reconcile` reaches it, so behavior is safe, but the rule as written is broader than the code.

## Rejected

- lens:code-correctness F2, `sparseSeed` still prunes unknown keys from a prime or legacy seed: the path only seeds an absent hub file from a copy and removes nothing from an existing committed file, so the Goal's failure (a binary stripping a committed key) cannot occur there.
- lens:code-affected-set F3, the "Local files" line in the background file is stale: the local layer was already pruned of every unknown key before this diff, so the imprecision is not introduced by it.
- lens:code-correctness item 8, indent 2 versus 4 in the `DropRetired` output: it only changes bytes on a rewrite that was already happening, and a rerun converges.
- `logResult` gaining `preserved` has no test: a log field, low value; the missing log for preserved-only runs is covered by F4.
