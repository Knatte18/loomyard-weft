---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, the reconcile criterion)
    summary: The reconcile criterion sets a standalone worktree file under a hub file, a stack that cannot exist, and read as a hub worktree file it claims a drop no code path performs
    fix: >-
      In decision-record.md's Acceptance criteria, replace the bullet starting "Under a hub file with `lens_set: csharp`, reconciling the hub file and a standalone worktree file" with two bullets — "Under a hub file with `lens_set: csharp`, `lyx config reconcile --layer default --apply` drops a hub-file lens entry equal to the csharp prose and keeps one equal to the base prose, and the loaded document is the same before and after." and "In a standalone worktree whose file holds `lens_set: csharp`, `lyx config reconcile --apply` drops a lens entry equal to the csharp prose and keeps one equal to the base prose, and the loaded document is the same before and after."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D3)
    summary: In a hub worktree, lyx config lenses --set lens_set=<name> writes the worktree layer and is rolled back with D3's refusal followed by the CLI's generic way forward to --set the key to a value it takes, which no value satisfies there
    fix: >-
      In D3, after the sentence on the hub-worktree load refusal, add "`configengine.Set` and the worktree editor's save refuse a write of the selector key to any layer above the base before anything is written, with the same refusal and way forward, so `lyx config lenses --set lens_set=<name>` in a hub worktree gives that refusal alone." and add the acceptance criterion "In a hub worktree, `lyx config lenses --set lens_set=csharp` writes nothing and is refused naming `--layer default` from the prime, with no other way forward."
---

### [MEDIUM] Reconcile criterion names a stack that cannot exist

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, the bullet "Under a hub file with `lens_set: csharp`, reconciling the hub file and a standalone worktree file each drops an entry equal to the csharp prose and keeps one equal to the base prose".

**Issue:** A standalone worktree has no hub file (`configengine.Dirs` with no `Board`), so "a standalone worktree file under a hub file" is no setup a test can build.
The other reading, a hub worktree file under that hub file, claims a drop no code path makes: `ReconcileAll` sends a hub worktree file to `reconcileSparseFile` in `internal/configsync/configsync.go`, which migrates and prunes and strips only under the one-time `pendingMigration`; only `reconcileBaseWorktreeFile` (standalone, through `yamlengine.Reconcile`'s `dropped`) and `ReconcileHubWideAt` (hub file, through `DropTemplateEqualNames`) drop template-equal names.
D4's rationale states the right pair ("in the hub file and in a standalone worktree file alike; a hub worktree file is only pruned"), so the criterion disagrees with D4.
A plan writer taking the criterion literally either writes an unbuildable test or adds a template-equal drop to `reconcileSparseFile`, which D4 does not decide.
Round 1's F3 fix swapped "worktree file" for "standalone worktree file" in this bullet but kept its "Under a hub file" opening, so the fix is incomplete rather than a new defect.

### [LOW] Hub-worktree `--set lens_set` gives two contradictory ways forward

**Location:** `_lyx/discussion/decision-record.md`, D3.

**Issue:** `lyx config lenses --set` with no `--layer` writes `configengine.LayerWorktree` through `setChecked` in `internal/configcli/configcli.go`, which runs `configengine.Set` (no selector check) and then `CheckLoad`.
Under D3 the load now fails with the hub-worktree refusal, so `newLoadFailure` rolls the file back and appends "way forward: --set the key to a value it takes".
No value of `lens_set` loads from a worktree file above a hub base, so the appended clause contradicts D3's own way forward, and D3's "remove `lens_set` from the named file" is moot because the rollback already did.
This is the most likely way an operator in a task worktree first meets the key, and D3 does not say what that write reports.

## Focus verdicts

- **D5, where a `remove` entry naming no lens is logged and how often.**
  Not a defect.
  `lenses.Load` keeps its two imports (`internal/configengine`, `gopkg.in/yaml.v3`, pinned by `internal/lenses/leaf_enforcement_test.go`) because the Warn goes through a configengine export, and the stated dedup, once per process per file and entry list, matches `warnUnknownKeys`'s `warnedUnknownKeys` key of file plus key list in `internal/configengine/config.go`.
  The named file is derivable: `remove` is a list a higher layer replaces whole, so the topmost layer holding it is its source, which configengine's own layer reader can name.
  The bound holds: `remove` is Guard, `Layers` refuses it in a local file through `ValidateLocal`, and `internal/configreg/runsettings.go` runs `ValidateLocal` over run settings.
- **D4, standalone worktree with no `lenses.yaml`.**
  Not a defect.
  `Set` scaffolds through `scaffoldIfMissing(path, …, string(spec.Template))` and then `yamlengine.SetValues`, which keeps every per-name name not equal to the template it is given; under an effective csharp template the five base-prose entries would stay pinned.
  D4's third bullet closes that path for `Set` and `Edit` (`BaseScaffold` returns the full template standalone), and standalone reconcile on an absent file already writes empty per-name maps (`mergePerNameMaps` leaves the merged mapping empty and reports `added`).
  The hub editor's `BaseScaffold` call in `internal/configcli/hubwide.go` gets `SparseBaseHeader`, so it pins nothing.
- **D3, a `lens_set` key in a worktree file above a hub base.**
  The worktree editor path (`configengine.Edit` at `LayerWorktree`, then fabric sync, no load check) and a tracked-file merge both reach the state; D3 bounds it by the load refusal, which names the file and the two-step way forward, and that way forward does remove the worktree key before the base write, so the base write does not leave it in place.
  `setChecked`'s before/after comparison would let a `--layer default` write through even with the worktree key still present, which is why the removal step comes first; D3 orders it so.
  The local layer gives `ValidateLocal`'s Guard refusal, since `Layers` checks a local file as it reads it, ahead of any selector check, as D3 states.
  The one gap is the hub-worktree `--set` message, F2.
- **Round-1 fix text.**
  The new D3 local-layer sentence, D4's scaffold bullet and D5's Warn helper each introduce no unbounded verb or bypass and leave no fact to be found later.
  The one criterion that claims a reconcile drop for a hub worktree file is F1.
