```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff changes on concrete inputs.

**Cases run:**
- **`fabricengine.ConfigSpec` → `configengine.MachineKeys`.**
  `Categories{branch_prefix: machine, pathspec: cross-pair, warp_exclude: cross-pair}` gives `["branch_prefix"]`, so `LayerFile(LayerMachine)` applies for fabric when the dirs carry both Anchor and Board. Correct.
- **`LocalDenied`.**
  `branch_prefix` (machine) is still not None, so it stays denied. The local layer still refuses it. Correct.
- **`configcli` `selectedLayer`, fabric, `--layer machine`.**
  `MachineKeys` is non-empty, so neither refusal case fires and the machine layer is chosen. Correct.
  `--layer machine --set pathspec=x` reaches the generic "is not a machine key" refusal (`configengine/layers.go:397`), whose message names the hub file and the local layer. Correct.
- **`fabricengine.LoadConfig` with `Dirs{Anchor, Board}`.**
  The hub file has `branch_prefix: hub/`, `pathspec: _lym` and the machine file has `branch_prefix: machine/`. The result is `machine/`, `_lym`. Correct.
  An empty machine value (`branch_prefix: ""`) overrides `hub/` with an empty prefix, which is the sparse-layer meaning. Correct.
  With no machine file the hub value is used. Correct.
  A missing `_lym/` still maps to the not-initialized message. Unchanged and correct.
- **`fabricengine.LoadConfig` with `Dirs{Board}` only** (the hub-file check, `configcli/hubwide.go:329`).
  No machine layer is stacked, so the check judges the hub file alone. Correct.
- **Every `LoadConfig(ConfigDirs(l))` caller.**
  `ConfigDirs` always sets Anchor, and Board whenever Bolt exists, so on one machine every reader in one hub checkout sees the same machine value. Correct.
- **`slugOfBranch` and `pairBranchNames` on a branch made under another machine's prefix.**
  Branch `a/foo` read on a machine with prefix `b/` gives slug `a/foo`.
  - In `keptReason`, `hasWeftEvidence("a/foo", …)` misses the `archive/foo/*` tags, so the branch is kept. Fails safe.
  - In `TaskBranchSlugs`, `a/foo` is listed as a held slug and missed by a fix-prefix match. Same behaviour as a hub-wide prefix change before this diff.
- **`fabricengine.Add` and the re-adopt seam (`battencli/readopt.go:73-80`) after ownership moves from machine A (`a/`) to machine B (`b/`).** Wrong result: see the finding below.

**Findings**

1. **MEDIUM, design. A pair cannot be re-adopted on a machine with a different prefix.**
   - **Location:** `internal/fabricengine/config.go:49` (`branch_prefix` → `CategoryMachine`); its effect shows in `internal/fabricengine/add.go:229` and `internal/battencli/readopt.go:79`.
   - **What happens:** `Add` builds the pair's branch as `t.cfg.BranchPrefix + slug` (`add.go:229-230`).
     The re-adopt seam calls `Add` with `AdoptOnly` on the current machine's config, and its `TaskBranch` seam goes through `PairTaskBranchOrOrigin`, which tries only the current prefix's name and the bare slug (`pairbranch.go:20-24`).
     So a gone run whose pair was made on machine A as `a/foo` and is re-adopted on machine B, with prefix `b/`, looks for `b/foo` and `b/foo-weft`.
     It finds neither locally or on origin and stops at the `AdoptOnly && !warpAdopted && warpAdoptTip == ""` branch (`add.go:460`). The live pair `a/foo` on origin is never found.
   - **Why the diff causes it:** with one hub-wide prefix this could happen only when the operator changed the prefix. The diff makes it the normal outcome of every owner move between two machines whose prefixes differ.
   - **Gap in the change description:** `_lym/landing/summary.md` "Open risks" names only the `PairTaskBranchOrOrigin` lookup gap. It does not name `Add` or re-adopt.
   - **Evidence:** `internal/fabricengine/add.go:229-230,460`, `internal/battencli/readopt.go:53-60,73-80`, `internal/fabricengine/pairbranch.go:20-24,56-69`.

No other wrong result, swallowed error or inconsistent failure state came up in the changed code.

**Tool calls:**
1. `add.go:195-260` and `beamcli/deps.go:270-300`: how `Add` names the warp and weft branches and what `HeldSlugs` does with `TaskBranchSlugs`. Context had only the single `add.go:229` grep line.
2. A grep for `AdoptOnly: true` callers and owner-machine code: to find the path that re-adopts a pair on another machine. That path was not in the manifest.
3. `battencli/readopt.go:40-95` plus a grep of `owner.go`: the re-adopt seams and owner claim. These were one hop beyond the manifest.
4. A grep for `AdoptOnly` in `add.go`: how `Add` behaves under `AdoptOnly` when no warp branch exists locally or on origin.
