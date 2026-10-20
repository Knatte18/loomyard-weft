---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: D1 rests on the premise that lyx config rolls back any write LoadConfig newly refuses, but the hub-wide editor path checks only configengine.Load, and D1 routes tree removal and malformed-key repair through that editor
    fix: In D1, replace the sentence at line 47 so it states that both hub-wide write paths of lyx config darn check the result with darn's own loader and roll back a new failure, and add a Decision bullet that copyStagedToHub in internal/configcli/hubwide.go gains the same module.CheckLoad before/after check setHubWide runs; add an acceptance criterion that an editor save leaving tree_scope_verify set with an emptied tree_scope, or holding tools and tools/, is refused and leaves the board file unchanged.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:59
    summary: D2 does not say the diff read reports raw paths, and D1's tree-key refusals miss keys git diff --name-only output never matches, so a tree can be silently inert
    fix: In D2, after the sentence at line 59, add that the read reports each path verbatim, as -z does, never core.quotePath's C-quoted form, so a non-ASCII path under a tree lies in that tree. In D1's refusal list at line 45, add a tree key holding an empty segment, such as tools//x, and state that only one trailing / is trimmed, so tools// is refused as an empty segment. Add both cases to the malformed-key acceptance criterion.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:44
    summary: The no-{targets} refusal reads as covering an empty tree_scope_verify, which would refuse the shipped template and the off command D1 itself names
    fix: At line 44, change the bullet to refuse a non-empty tree_scope_verify without {targets}, so the empty value the template ships and lyx config darn --set tree_scope_verify= writes stays accepted.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:50
    summary: D1 says a tree is removed by editing the map but not that removing the last tree while tree_scope_verify is set is refused, so the off sequence for a hub dropping tree scope entirely is unstated
    fix: At line 50, append that removing the last tree requires turning tree scope off first with lyx config darn --set tree_scope_verify=, or clearing tree_scope_verify in the same editor save.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:113
    summary: D5 names a way forward only for the three GuardScanError kinds, while FindGuardTests also fails on a directory it cannot read, which carries no file and line
    fix: At line 113, add that a scan failure that is not a guard-scan error, such as an unreadable package directory or a failed package enumeration, refuses the verb before any slot with the error and the directory it names, and that only the three content kinds name a file and line.
---

### [MEDIUM] The editor write path does not run darn's loader

**Location:** `_lyx/discussion/decision-record.md:47`

**Issue:** D1 states "`lyx config` rolls back a write after which `LoadConfig` fails with a new error" and builds the on/off sequence on it.
That holds for `--set`: `setHubWide` in `internal/configcli/hubwide.go` runs `module.CheckLoad` before and after and rolls back through `newLoadFailure`.
The editor path does not: `editHubWide` calls `copyStagedToHub`, which validates only through `validateHubFile`, a bare `configengine.Load` that never calls `darnengine.LoadConfig`.
D1 sends the operator to that editor to remove a tree and to repair a malformed or duplicate key.
An editor save that empties `tree_scope` while `tree_scope_verify` is set, or that holds `tools` and `tools/`, is committed to Bolt unchecked.
`VerifyCommand` loads `LoadConfig` at each call, so every in-flight darn run of the hub then fails at its next gate.
The record's premise is half false, and no Decision makes the editor path match it.

### [MEDIUM] Diff paths can come back quoted, and some tree keys never match

**Location:** `_lyx/discussion/decision-record.md:59`

**Issue:** `ChangedPaths` in `internal/fabricengine/parenttips.go` reads through `diffNames` in `internal/fabricengine/configchanges.go`, `git diff --name-only` without `-z`.
Git C-quotes a path outside core.quotePath's ASCII set, so `docs/ä.md` comes back as `"docs/\303\244.md"`, which lies in no tree.
`internal/gitrepo` already treats that quoted form as a defect in its own reads.
D2 asks the read to change for renames but says nothing about quoting, so a docs-only diff with a non-ASCII file classifies as production, and a non-ASCII tree key never matches.
D1's refusal list also lets through a key with an empty segment, such as `tools//x`, which no git path matches.
It also leaves open whether `tools//` trims to `tools/` or `tools`.
Each such key is accepted and silently inert.
The failure direction is safe, as the diff counts as production, so this is not BLOCKING.

### [LOW] The no-`{targets}` refusal covers the empty value

**Location:** `_lyx/discussion/decision-record.md:44`

**Issue:** The refusal list names "a `tree_scope_verify` without `{targets}`" with no non-empty qualifier.
Read literally, it refuses the empty value the template ships and the `--set tree_scope_verify=` off command at line 49.
"Tree scope is on only when both are non-empty" implies the intended reading, but a refusal rule should state its own condition.

### [LOW] Removing the last tree is refused while scope is on

**Location:** `_lyx/discussion/decision-record.md:50`

**Issue:** D1's first refusal rejects a `tree_scope_verify` set while `tree_scope` holds no tree.
An operator who removes every tree in the editor while `tree_scope_verify` is set hits that refusal, once F1's check runs on the editor path.
The record names the on and off commands but not this ordering.

### [LOW] D5 leaves the non-content scan error unstated

**Location:** `_lyx/discussion/decision-record.md:113`

**Issue:** `FindGuardTests` in `internal/impactset/guards.go` returns `GuardScanError` for the three content kinds D5 lists.
It also returns a plain wrapped error when `os.ReadDir` of a package directory fails, which names no file or line.
D5 says a scan error names "the file and line", which holds only for the content kinds.
The verb's behaviour on the other kind, and on a failed package enumeration, is unstated.

## Verdicts on the focus directives

- Focus 1: `tree_scope` as a `ConfigOpenMaps` entry is carried whole by `Reconcile` and `FillMissing` per `internal/yamlengine/doc.go`.
  `MigrateConfig`'s `RenameKeys` touches only the retired fan keys.
  `setOpenMapEntry` in `internal/yamlengine/set.go` turns a null or `{}` template value into a mapping, so `--set tree_scope.<tree>=<target>` works with either empty template spelling, including an empty target.
  The `--set` on and off sequences hold against `setHubWide`'s rollback; the editor path does not, which is F1.
- Focus 2: D4 classifies at each Publish call, after the moved-parent check, through a seam separate from `Config.PublishVerify`.
  The darn recipe routes a PR-Gate rejection through Darn-Write, Describe and Merge-In before Publish, so a rework that adds a production path gets Merge-In's whole `verify` and `publish_verify` at the next Publish, as an acceptance criterion pins.
  Not a defect.
- Focus 3: the categories match `ConfigSpec` (`verify` is `CategoryExecutable`) and `coveringCategory` in `internal/configengine/layers.go`, which covers every path under an open map's entry.
  The tree-key list has the gaps in F2.
- Focus 4: D5 states the scanned directories and the content-kind refusals; the non-content kind is F5.
  D4's claim that the whole suite runs at Merge-In for every production diff, and at the writer gates wherever `impactset` falls back, agrees with `internal/impactset/doc.go`'s fallback for any changed non-`.md` file outside a Go package directory.
  Not a defect.
