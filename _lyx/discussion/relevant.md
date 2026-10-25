# Relevant files

- `contracts/recipes/loom-recipe.yaml` — the loom row order and edges the three rows join
- `contracts/recipes/darn-recipe.yaml` — the darn row order, and darn's `Describe` entry-gate precedent
- `internal/shedengine/entry.go` `Shed.enter` — how an entry gate that does not hold records Skipped and routes by OnDone
- `internal/shedbuild/entrygate.go` `parseEntryGate` — the entry_gate check syntax a `drift.enabled` gate uses
- `internal/shedrecipe/entries_command.go` `commandEntry` — a registry entry reading an executable command key
- `internal/configreg/configreg.go` — where the hub-wide `drift` module registers
- `internal/gateslot/doc.go` — the non-tier slot holders and the gate keys the template lists
- `internal/gateslot/template.yaml` — gate keys, beside which `drift-verify` goes
- `internal/runproc/doc.go` — process records, roles and the end sequence the capture child joins
- `internal/runproc/record.go` — the role constants a `capture` role joins
- `internal/pairteardown/teardown.go` `Teardown.endOwnedProcesses` — ends a pair's processes, capture included
- `internal/loomengine/template.yaml` — model-spec key style the drift model keys follow
- `pattern/PATTERN-gate-slot.md` — slot holders and verify sites
- `pattern/PATTERN-run-process-owner.md` — registering and ending the detached capture
- `pattern/PATTERN-shed-recipe-registry.md` — entry gates and registry entries for the new rows
