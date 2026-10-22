---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D5 (skipped and logged at Warn naming the entry)
    summary: D5 has lenses.Load log a Warn, but internal/lenses may import only the standard library, configengine and yaml (PATTERN-leaf-packages), and the Warn as specified repeats on every load
    fix: In D5, replace "is skipped and logged at Warn naming the entry" with a mechanism that keeps internal/lenses inside its import set, e.g. "is skipped; configengine logs it at Warn through an exported helper the lenses loader calls, once per process per file and entry list, the way warnUnknownKeys dedups". Add a Constraints bullet "PATTERN-leaf-packages — internal/lenses gains no import; the remove Warn is logged through configengine". Update the matching acceptance criterion to "A remove entry naming no lens loads, logged at Warn once per process".
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D4 (Only paths that list template keys or scaffold a whole template keep the base template)
    summary: A standalone base scaffolded from the whole base template pins the base prose of every lens, so a lens_set picked through Set or Edit on an absent file never takes effect for the five overlaid lenses
    fix: In D4, add the rule that a scaffold of a base file for a module with an overlay writes its per-name maps empty, as standalone reconcile already writes a fresh file, so no scaffold pins a template entry. Add an acceptance criterion "In a standalone worktree with no lenses.yaml, lyx config lenses --set lens_set=csharp resolves the five lenses to the csharp prose".
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D4 Rationale, and the acceptance criterion beginning "Under a hub file with lens_set csharp, reconciling the hub file and a worktree file"
    summary: No reconcile of a worktree file in a hub drops template-equal per-name entries, and no Decision adds that drop, so the rationale and criterion claim behaviour no path has
    fix: Narrow both to what the code does. In D4's Rationale, replace "in the hub file and in a worktree file alike" with "in the hub file and in a standalone worktree file alike; a hub worktree file is only pruned, and the one-time migration strip judges it against the layers below it". In the criterion, replace "reconciling the hub file and a worktree file each drops" with "reconciling the hub file and a standalone worktree file each drops".
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3 (the refusal of a lens_set key above the base and its way forward)
    summary: The way forward D3 names for a lens_set above the base writes the base file but leaves the worktree key in place, so the refusal persists; the worktree editor path, with no load check, and a tracked-file merge both reach that state
    fix: In D3, extend the refusal's way forward to "remove lens_set from <the named file>, then set it in the base with lyx config lenses --layer default --set lens_set=<name> from the hub's prime worktree", and add the removal step to the matching acceptance criterion.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md acceptance criterion "a lens_set key in a worktree or local layer above a hub base is refused naming the file and --layer default"
    summary: Because lens_set is a Guard key, ValidateLocal refuses a local-layer lens_set first, with the local-override message that names neither the base file nor --layer default, so D3's refusal never fires for the local layer
    fix: Restrict the criterion's D3 message to the worktree layer, e.g. "a lens_set key in a worktree file above a hub base is refused naming the file and --layer default; one in a local file is refused as a guard key", and say in D3 that the local layer keeps the guard-key refusal.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md acceptance criterion "each layer fan override holds its base fan's lenses in order"
    summary: D2 requires a layer fan to start with the base fan's lenses in base order, but the criterion checks only a subsequence, which passes after a base edit removes a lens the layer fan still runs
    fix: Change the criterion to "each layer fan override starts with its base fan's lenses, exactly and in base order, followed only by the layer's additions".
---

### [BLOCKING] D5's Warn breaks the lenses leaf import set and repeats on every load

**Location:** `_lyx/discussion/decision-record.md` D5.
**Issue:** D5 has `lenses.Load` skip a `remove` entry that names no lens and log it at Warn.
`PATTERN-leaf-packages` limits `internal/lenses` to the standard library, `internal/configengine` and `gopkg.in/yaml.v3`, and `internal/lenses/leaf_enforcement_test.go:12-19` pins that set.
Logging goes through `internal/logger` (`internal/logger/logger.go`, a slog wrapper whose callers never use slog directly), which lenses may not import.
The Warn as written therefore cannot be built without breaking a listed constraint.
Separately, `lenses.Load` runs at every lens consumer's load: burler wiring, `twillLenses` in each round, and loom's and darn's config loads through `ValidateFanKeys`.
An unknown `remove` entry is a standing state, so the Warn repeats on every load.
The logger's Level policy (`internal/logger/logger.go:140-152`) and the precedent of `warnUnknownKeys` (`internal/configengine/config.go:263-305`) both log a standing config condition once.
The existing-paths and walkthrough forks raised the repetition, and this finding merges it.

### [MEDIUM] A standalone base scaffold pins base prose and defeats the picked layer

**Location:** `_lyx/discussion/decision-record.md` D4, with D2 and D3's standalone way forward.
**Issue:** D4 keeps the base template for "paths that … scaffold a whole template".
Take a standalone worktree with no `lenses.yaml` and run `lyx config lenses --set lens_set=csharp`, D3's standalone way forward.
`Set` scaffolds the full base template through `scaffoldIfMissing` (`internal/configengine/set.go:58`).
`SetValues` then drops only the names equal to the template it is handed (`internal/yamlengine/set.go:70`), which under D4 is the effective csharp template.
The five base-prose entries differ from the csharp prose, so they stay in the file.
A file entry beats the layer, so those lenses resolve to base prose.
Reconcile keeps them too, because D4's rationale keeps an entry equal to the base prose.
`Edit` scaffolds the same way through `BaseScaffold` (`internal/configengine/edit.go:107-112`).
The existing-paths and bound-breaker forks raised the same defect.

### [MEDIUM] The worktree-file template-equal drop in a hub does not exist

**Location:** `_lyx/discussion/decision-record.md` D4 Rationale and the reconcile acceptance criterion.
**Issue:** In a hub, `ReconcileAll` sends a worktree file to `reconcileSparseFile` (`internal/configsync/configsync.go:399`).
That function only migrates and prunes (`:548`), and it runs the below-layer strip only while `pendingMigration` is due (`:441-455`, `:556`).
`DropTemplateEqualNames` runs only on board files (`:917`), and `yamlengine.Reconcile`'s drop runs only on a standalone base (`:498`).
D4 only points existing judging paths at the effective template, so no Decision adds the drop.
A plan writer would either build an unrequested drop or write a test that cannot pass.
All four forks raised this.

### [MEDIUM] D3's way forward does not clear its own refusal

**Location:** `_lyx/discussion/decision-record.md` D3.
**Issue:** The refusal fires when a `lens_set` key is present above the base, whatever its value.
Running the named way forward, `--layer default --set lens_set=<name>`, writes the board file through `setHubWide`, which checks `CheckLoad` over `Dirs{Board}` alone (`internal/configcli/hubwide.go:347-361`), and so succeeds.
The worktree key stays, so every load from that worktree is still refused.
No later reconcile removes the key once the sparse marker exists (`internal/configsync/configsync.go:441-455`).
A worktree `--set` is rolled back by `setChecked`, but the worktree editor path runs `configengine.Edit` with no load check (`internal/configcli/configcli.go` `editOne`, `internal/configengine/edit.go:120-133`).
A merge of the tracked file is a second way in.
The interactions, existing-paths and bound-breaker forks raised the same defect.

### [LOW] A local-layer lens_set is refused with the guard message, not D3's

**Location:** acceptance criterion on refusing `lens_set` above a hub base.
**Issue:** `readLayers` validates the local file with `ValidateLocal` (`internal/configengine/layers.go:424-431`).
Because `lens_set` is classified Guard, that check refuses it with `key "lens_set" is guard and cannot be set in a local override` (`:290-296`).
That message names neither the base file nor `--layer default`.
The existing-paths and bound-breaker forks raised the same point.

### [LOW] The superset acceptance criterion is weaker than D2's rule

**Location:** acceptance criterion "each layer fan override holds its base fan's lenses in order".
**Issue:** D2 requires every lens of the base fan, in base order, followed by the layer's additions: a prefix.
The criterion checks only a subsequence.
Suppose a base edit removes a lens from `review-code` that the csharp layer's `review-code` still runs: the criterion passes.
That is exactly the drift D2's rationale says the rule prevents.

## Focus directive verdicts

- D3 and D6 way forward: `lyx config lenses --layer default --set lens_set=<name>` from the prime is a real invocation.
  `selectedLayer` maps `default` to `LayerDefault` for the per-worktree `lenses` module, and `hubWriteRefusal` passes a slug-free session on the prime.
  `setHubWide` writes, validates and checks the load, then commits.
  A Guard key is refused only by the local layer's `ValidateLocal`, so the default layer takes it.
  This part is not a defect; the refusal path has the gap in F4.
- D4 and D5 bounds: D5 states the `remove` bound, the worktree file's `remove` replacing the hub's whole included.
  D6 states the interim neutral base for Go hubs, with the reason it is accepted.
  The defects found here are F1 and F4.
- D4 path classification: one path is ambiguous, scaffolding (F2).
  Under D4's wording, `Provenance` and below-layer composition clearly use the effective template.
- Decision record versus support log: no requirement the plan writer needs is found only in the support log.
  The stale Interview turns, the key named `language` and a no-lens `remove` refused, are turn-by-turn history.
  The lens-review bullets and the Question ledger record the later reversals, and Plan-Write never reads the support log, so this is not a defect.
- Deferred facts: the cited `PATTERN-*` ids and `TestModules_EveryTemplateKeyClassified` exist.
  The NORCE guard and try/catch rule is in `dotnet-verify-protocol`'s body.
  The exact csharp and python prose is content the implementer writes from D2's topic lists, not a fact the design rests on.

## Rejected

- walkthrough F3, that `Config` must record the fans `remove` emptied and gain the `lens_set` and `remove` fields: that is decode shape and interface detail the writer was told not to settle, and D5 already states the observable refusal.
