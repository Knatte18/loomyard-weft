---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (third paragraph and Bound)
    summary: The gate load failure that names the machine file offers only prime steps, so a pair session reading it has no way forward of its own, and the unguarded file removal it names invites the pair to take it.
    fix: In D3, after the sentence naming both ways forward from the prime, add that the message also names the step for a session that is not the prime, in the wording internal/gatecli/run.go already uses for ErrConfigAbsent (a session lyx refuses the verb from reports status FAILED and the orch runs it), so a pair reports instead of removing the file. Add to D3's Bound that this clause is what keeps a pair from taking the removal step. Extend the acceptance criterion on a machine file at go_parallel 0 making lyx gate test fail to require that clause in the message.
---

### [MEDIUM] Gate load failure names no way forward for the pair that reads it

**Location:** `_lyx/discussion/decision-record.md` D3, third paragraph and its Bound; D4.

**Issue:** The sessions that hit a gate load failure are mostly pair agents running `lyx gate test` in their own task worktree.
The `lyx gate test` limits read in `internal/gatecli/run.go` runs from whatever worktree calls it, and `hubgeom.GateSlots` loads through `fabricengine.ConfigDirs`, whose `Anchor` is always the caller's anchor.
D3's message names two ways forward, each "from the prime": `lyx config gate --layer machine` and removing the file.
From a pair, the first is refused by D4's `hubWriteRefusal` guard.
The second works from a pair: D3's Bound says D4's guard does not stop a pair removing it, and the shell guard rows in `internal/shellguard/rows.go` refuse only an `rm` on a computed path, not a literal one.
So the message gives a pair reader no step it may take, and names one step it can take, without saying that it must not.
Removing the file returns every gate run of the hub on that machine to the hub file's values, the CPU budget the operator lowered the machine below.
The bound D3 states covers what the removal lets through, but not that the message itself steers a pair towards it.
The repo already has the pattern for this: the `ErrConfigAbsent` message in `internal/gatecli/run.go` adds "a session lyx refuses the verb from reports status: FAILED and the orch runs it".
D3 never asks for that clause, and no acceptance criterion pins one.

## Rejected

- D1, machine-layer stacking for an anchor with no `_lyx/` (focus 1): not a defect.
  `load` in `internal/configengine/config.go` and `Provenance` in `internal/configengine/layers.go` hand `Layers` a copy of `Dirs` with `Anchor` cleared, so a rule keyed on the cleared copy would drop the machine layer.
  D1 says the rule reads the told `Dirs`, the Notes for the plan writer repeat that it is judged before the clearing, and the acceptance criterion on a worktree whose anchor has no `_lyx/` pins both the load and `--resolved`.
  `fabricengine.ConfigDirs` always sets `Anchor` from `l.AnchorPath()` and sets `Board` whenever the board dir exists, the local layer refuses `machine` keys per D2, and a hub-wide module has no worktree layer when `Board` is set.
  So every worktree of one hub on one machine stacks template, hub file and the same machine file for `slots` and `go_parallel`, and resolves the same values; the one exception, two lyx builds, is in Open risks.
- D3 and D4, ways forward from the prime (focus 2): both steps work from the prime.
  Neither `lyx config` nor `rm` on a literal path is in a shell-guard row, so `PATTERN-no-denied-recovery` holds.
  D3 states why removal is named (`--set` cannot remove a refused key, the editor needs a terminal), and its Bound states what removal lets through and what keeps the result valid.
  The pair-side gap is F1.
- D2 and D5, a retired key in the machine file (focus 3): one outcome.
  `Layers` validates each layer before `dropRetired` runs in `load`; D2 admits a retired key by its recorded category under the machine rule, so a retired `machine` key passes validation and is then dropped, and any other retired key is refused at validation.
  D5 states the same split, and `warnUnknownKeys` covers only the default and worktree layers, so "with no warning" matches the code.
- Hub-file checks and the failure message (focus 3): stated.
  `validateHubFile`, `setHubWide` and `copyStagedToHub` in `internal/configcli/hubwide.go` all load `Dirs{Board}` alone; D1 says such a load stacks no machine layer, D3 says such a load's message names the hub file only, and the second acceptance criterion pins both for `--set go_parallel=0`.
- D5's "and nothing else" leaves out that `lyx config gate --print` and `--resolved` also fail on a refused machine key, through `configengine.Load`.
  That error names the file and the key, and the sentence contrasts the outcome with `Ensure` blocking every run, so it misleads no plan writer; not raised.
- Board-entry lock: no Decision edits the task's own board entry, and Q10 leaves the other entry `xdr-git-test-file-burst` to the orch.
