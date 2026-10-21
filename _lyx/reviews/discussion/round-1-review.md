---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D1
    summary: D1's both-Board-and-Anchor rule leaves open whether it is judged on the told Dirs or on load's layerDirs, whose Anchor load clears for an anchor without _lyx/, so such a worktree would drop the machine layer and disagree on the pool size
    fix: In D1, after "It applies only when the spec is `HubWide` and both `Dirs.Board` and `Dirs.Anchor` are set", add one sentence fixing the rule on the told dirs, for example "The rule reads the told `Dirs`, so a worktree whose anchor has no `_lyx/`, for which `load` and `Provenance` drop the worktree and local layers, still stacks the machine layer, since the file belongs to the board checkout, not the worktree." Add an acceptance criterion - "A gate load from a worktree whose anchor has no `_lyx/` resolves the machine file's `go_parallel`, and `--resolved` names `machine` for it."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3
    summary: D3 makes every gate load failure message name the machine file and its ways forward, but the Board-only hub-file checks D1 keeps machine-free run the same gate loader through CheckLoad, so a refused hub-file write would point the operator at a machine file that played no part
    fix: In D3, replace "Every gate load failure message names the hub file and this machine's file and both ways forward" with "A gate load failure from a stack that holds the machine layer names the hub file and this machine's file and both ways forward; a load with `Board` alone, as the hub-file checks make, names the hub file and its way forward only." Extend the second acceptance criterion - "and the refusal names neither the machine file nor `--layer machine`."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2, D5
    summary: D5 says every load drops retired keys from the machine file so a retired key still loads, but D2 admits a retired key only when its recorded category is machine and the layer check runs before the drop, so a stale key has two stated outcomes and the silent drop of a retired machine key is unstated
    fix: In D5, replace "Every load drops retired keys from it, as from every layer, so a key retired with its category recorded still loads." with "A retired key whose recorded category is `machine` is admitted and dropped by every load, with no warning, so its value stops applying; any other retired key is refused by D2 and fails the gate load with the D3 message." Add an acceptance criterion - "A machine file holding `cli_wait_sec` fails the gate load naming the file and the key."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3
    summary: D3's ways forward name lyx config gate --layer machine without from the prime, which D4 refuses from the pair sessions that mostly read gate failures, and introduce removing the file as an escape hatch with no stated bound
    fix: In D3, write the machine-file way forward as "`lyx config gate --layer machine` from the prime, or removing the file at its path", and add a Bound paragraph - "Bound. Removing the file returns every gate run of the hub on this machine to the hub file's values, which the hub-file checks keep valid; like a direct write, D4's guard does not stop a pair removing it." Extend the `lyx gate test` acceptance criterion to require "from the prime" in the message.
  - id: F5
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1, D5
    summary: D1 gives every hub-wide module a machine layer, so a stray fabric, landing, darn, board or profiles machine file is refused at every Anchor-and-Board load of that module on the machine, contradicting D5's a refused key fails the gate load and nothing else
    fix: In D5, replace "A key the machine layer refuses fails the gate load with the D3 message, and nothing else." with "A key the machine layer refuses fails that module's loads on this machine; for gate the failure carries the D3 message, and for a hub-wide module with no `machine` key any present unit is refused, a state only a direct file write reaches."
  - id: F6
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4
    summary: D4's a refused key is refused before the write holds for --set only; the editor path writes the file first, and today configengine.Edit validates no layer but local and editLocal runs no CheckLoad, so the editor save's outcome is unstated
    fix: In D4, replace "A key the machine layer refuses is refused before the write, naming the key and the layers that take it." with "A `--set` with a key the machine layer refuses is refused before the write, naming the key and the layers that take it; an editor save is checked after the save by the same rule and by `CheckLoad`, and a refused one is put back to its previous bytes, or removed when the edit created it."
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2
    summary: D2's bound calls a direct machine-file write the default layer's existing bound, but a direct write to the hub file lands on Bolt at the next board sync while one under .lyx leaves no trace, and D2 states no upper bound on a raised slots or go_parallel
    fix: In D2, replace the Bound's second sentence with "What it lets through is a value written to the machine file, which the D4 write guard confines to the prime, and a direct file write, which that guard does not stop and which, unlike a direct hub-file write, never reaches Bolt or any record." and replace its third sentence with "The value still passes gate's loader at every acquire, so `slots` and `go_parallel` stay at least 1; no upper bound applies, as on the hub file."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4
    summary: D4 lists the menu among the places that accept --layer machine, but lyx config menu has no --layer flag and routes a hub-wide module to its hub file, so only the menu's mark can carry machine
    fix: In D4's first sentence, replace "for `--print`, `--set`, an editor edit and the menu" with "for `--print`, `--set` and an editor edit"; the existing sentence on the menu's `machine` mark stays as the menu's only change.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1 Rationale
    summary: D1's rationale places the editor staging copy at BoardDir/.lyx/config/module.yaml, but editHubWide stages under the worktree anchor's .lyx/config/, so the stated reason for the .machine.yaml name rests on a wrong path
    fix: In D1's Rationale, replace "The `.machine.yaml` name keeps the file apart from the editor staging copy at `<BoardDir>/.lyx/config/<module>.yaml`." with "The `.machine.yaml` name keeps the file apart from the `<module>.yaml` and `<module>.local.yaml` names `.lyx/config/` already uses."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Open risks
    summary: The gate-language-neutral risk says that move must carry the machine file's value, but key moves run only in reconcile, which D5 forbids from touching the machine file, and a retired machine key is dropped silently
    fix: In the third Open risks bullet, replace "that move must classify `parallel` as `machine` and carry the machine file's value, or the override is lost." with "that move must classify `parallel` as `machine`; carrying the machine file's value means amending D5, since only reconcile moves keys, and otherwise the retired `go_parallel` is dropped silently and the operator re-sets the override."
---

The record is a sound design over the existing layer machinery and fits its board entry.
D1's Board-only rule for hub-file checks holds at every check the code makes (`validateHubFile`, `setHubWide`, `copyStagedToHub`), and D5's claim that no reconcile lists `<BoardDir>/.lyx/config/` holds against `snapshotBoardConfig`, the `hubreconcile` walk and `configsync`.
The findings tighten applicability, the failure message's reach and the stated bounds.

### [MEDIUM] Machine-layer applicability is ambiguous for an anchor without `_lyx/`

**Location:** `_lyx/discussion/decision-record.md` D1

**Issue:** `load` in `internal/configengine/config.go` clears `layerDirs.Anchor` when the anchor has no `_lyx/` before `StackLayers`, and `Provenance` in `internal/configengine/layers.go` does the same.
Judged on those dirs, D1's "both `Dirs.Board` and `Dirs.Anchor` are set" drops the machine layer for such a worktree, so `GateSlots` there resolves the hub's `slots` while the hub's other worktrees on the machine resolve the override, breaking D1's own purpose that every worktree agrees on the pool size.
The record does not say which dirs the rule reads.
Raised by every lens.

### [MEDIUM] D3's message reaches the Board-only hub-file checks

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** The gate loader's way forward is appended inside `gateslot.LoadConfig` (`withConfigWayForward`), which `configreg`'s gate `CheckLoad` runs.
`setHubWide` and `copyStagedToHub` in `internal/configcli/hubwide.go` call that `CheckLoad` with `Dirs{Board}`, so under D3 as written a refused hub-file `--set go_parallel=0` would name the machine file and `--layer machine` as ways forward although D1 keeps that file out of the check, a wrong way forward under `PATTERN-refusal-way-forward`.

### [MEDIUM] Retired keys in the machine file have two stated outcomes

**Location:** `_lyx/discussion/decision-record.md` D2, D5

**Issue:** `load` validates each layer in `Layers` before `dropRetired` runs.
Under D2 a retired key is admitted only when its recorded category is `machine`, so `cli_wait_sec`, recorded `CategoryCrossPair` in `internal/gateslot/config.go`, is refused and fails the gate load, while D5 says every retired key is dropped and still loads.
A retired `machine` key is admitted and dropped with no warning, since `warnUnknownKeys` skips every layer but default and worktree; the record states neither outcome, and no criterion pins either.

### [MEDIUM] D3's ways forward lack "from the prime" and the removal hatch has no bound

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** `lyx gate test` failures are mostly read in pair sessions (`internal/gatecli/run.go`), where D4's `hubWriteRefusal` refuses `lyx config gate --layer machine`, so the first named way forward fails where the message is seen.
Removing the file is a new escape hatch, and the rubric's attack-surface question requires its Decision to say what it lets through and what bounds it; D3 says neither.

### [LOW] Every hub-wide module gains a machine layer, but D5 scopes the failure to gate

**Location:** `_lyx/discussion/decision-record.md` D1, D5

**Issue:** D1's rule applies to every `HubWide` spec, so `fabricengine.LoadConfig(ConfigDirs(l))` and the landing, darn, board and profiles loads read `<module>.machine.yaml` too, and since none of those modules classifies a key `machine`, any unit there is refused at every load on the machine.
D5's "and nothing else" is then false for a stray file.

### [LOW] The editor path's outcome for a refused machine key is unstated

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `configengine.Edit` in `internal/configengine/edit.go` hands a non-base, non-local layer to `EditPath` with no check and no put-back, and `editLocal` in `internal/configcli/configcli.go` runs no `CheckLoad`.
D4's "refused before the write" cannot hold for an editor save, and the record states no after-save check or rollback for it.

### [LOW] D2's bound misstates the direct-write bound and omits the upper bound

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** A direct write to the hub file is committed by the next board sync (`reportHubWrite`'s message in `internal/configcli/hubwide.go`), while a direct write under `.lyx` leaves no record, so "the default layer's existing bound" is not the same bound.
`decodeConfig` bounds only from below, and D2 never states that a raised value is uncapped.

### [LOW] D4 lists the menu as taking `--layer machine`

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** `menuCmd` in `internal/configcli/configcli.go` declares no flags, and `menu` calls `editOne` with an empty layer, which `selectedLayer` maps to the hub file for a hub-wide module.
Scope's "the menu's layer marks" is the only menu change any path supports.

### [LOW] D1's rationale names a wrong staging path

**Location:** `_lyx/discussion/decision-record.md` D1 Rationale

**Issue:** `editHubWide` stages at `configengine.StagingFile(dirs.worktree, module.Name)`, under the worktree anchor's `.lyx/config/`, not under the board dir's.
The `.machine.yaml` name still collides with nothing, so only the stated reason is wrong.

### [LOW] The `gate-language-neutral` risk contradicts D5

**Location:** `_lyx/discussion/decision-record.md` Open risks

**Issue:** `configreg.KeyMoves` are applied only by reconcile (`internal/configsync/movekeys.go`), which D5 forbids from reading or writing the machine file, so "carry the machine file's value" cannot happen without amending D5.

## Rejected

- Concurrent readers seeing a value `setChecked` later rolls back, or a torn read of the non-atomic `setSparse` write, fail another worktree's acquire (bound-breaker F7): the hub-file `--set` has the same write-then-check window over a file every acquire reads today, so the machine write adds no new class of exposure, and the record's "as every `--set` is" parity is accurate.
- `layerPath` checks `FindBaseDir(Anchor)` for every layer but default while the machine file lives under the board dir (existing-paths): an implementation detail for the plan, not a design defect in the record.
- The Board-only loads in `prepareRunSettings` and the executable `SettingLookup` (`internal/battencli/wire.go`) as unstated machine-free sites: they read no gate key, so the machine layer's absence there changes nothing.
