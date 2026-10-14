---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: contracts/recipes/batten-recipe.yaml:43
    summary: The Open-Window, Seed-IDE and Open-IDE rows sit after Seed-Child, where board scope item 2 puts them right after Worktree-Create, and no Added-after-Discussion entry records the move; the carried-over Darn-Review risk defers it to the parent
    fix: Escalate to the run's parent, who either records the after-Seed-Child placement with `lyx loom decision add` as an amendment of scope item 2, or refuses it, in which case the rows move to directly after Worktree-Create and the per-task flag reading is redesigned; nothing in the diff changes until the parent decides.
---

### [BLOCKING] Row placement needs the parent's decision

**Location:** contracts/recipes/batten-recipe.yaml:43
**Issue:** Board scope item 2 reads "Batten's recipe gets them as rows right after Worktree-Create, so the window still opens as soon as the pair exists."
The diff runs `Worktree-Create`, `Seed-Child`, `Open-Window`, `Seed-IDE`, `Open-IDE`, `Run-Shed`, so the rows follow `Seed-Child` instead.
The decision record has no `Added after Discussion` section, so no parent, operator or arbiter entry amends scope item 2; the placement rests only on the writer's own decision and an auto-mode assumption that says "the parent may move them".
The writer's reason holds against the code: `writeRunSettings` in `internal/battenshed/seamchild.go` is where a task's `run_settings` reach the pair's local layer, and scope item 3 requires those settings to set the flags per task, so the entry's two items pull against each other.
The Darn-Review carry-over in the decision record keeps this open as `row-placement-departs-from-board-entry`, and its review says the move has the effect of an unrecorded parent correction and is the parent's to settle; the fixer report defers it for the same reason.
Approving would take the writer's departure as the measure, and rejecting would refuse a choice the entry itself forces; only the parent can record or refuse the correction.

Everything else in the diff matches the amended entry:

- One `SwitchedProducer` in `internal/shedadapters`, registered in `internal/shedrecipe` under `OpenWindow`, `SeedIDE` and `OpenIDE`, so any recipe can take the rows (scope 1).
- `Seed-IDE` calls `ideengine.SeedTask`, which writes colour and `.vscode/` with the folder-open task chain and leaves `keybindings.json` alone; `Open-IDE` calls `ideengine.OpenTask` (scope 1, orch call).
- The window opening in the Worktree-Create seam and in Run-Shed, `OpenTerminalOnce` and its once-marker are removed (scope 2).
- `batten.yaml` in `internal/battenshed` has `open_window: true`, `seed_ide: true`, `open_ide: false`, all `CategoryNone`, so open to the local layer; `--window`'s hidden `--open-terminal` forces only `open_window` (scope 3).
- Fabric is not touched (scope 4).
- Docs: `internal/shedadapters`, `internal/battenshed`, `internal/battenrecipe` and `internal/ideengine` package docs, `docs/overview.md`'s batten entry, CLAUDE.md's "Watching runs" and `PATTERN-config-strictness` (scope 5).

The record's other open risk, that a plain `lyx batten run` now opens a window by default, follows from the entry's own `open_window` default of true and is no deviation.

## Overlaps

- `overview-seats` (row 12, Webster): text-only overlap in `internal/shedadapters/doc.go`, `internal/shedrecipe/registry.go`, `internal/shedrecipe/conformance_test.go`, `internal/testkit/envkit/envkit.go`, `internal/battencli/wire_seams_test.go`, `internal/loomcli/wiring_seams_test.go` and `docs/overview.md`.
  It replaces `SingleLLMProducer` with `MultiLLMProducer` and reworks the loom checklist, and its record puts the batten overview out of scope; no behaviour, decision or PATTERN entry is shared, since this run edits `PATTERN-config-strictness` and that run edits `PATTERN.md` and `PATTERN-agent-name`.
  Landing order: this run (row 22) lands first; `overview-seats` (row 12) takes its changes at Merge-In.
- `twill-fork0` (row 7, paused at Plan-Write): no changed paths, no overlap.
- Parent commits: none listed.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml`.

`shed-tackler` is not in flight; its board entry depends on this task and lists no conflicting decision.
