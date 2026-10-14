---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: contracts/recipes/batten-recipe.yaml:43
    summary: The three rows sit after Seed-Child, where the board entry asks for them right after Worktree-Create; no Added-after-Discussion entry licenses the move
    fix: Leave the code in place and defer this finding to the run's parent; recording or refusing the placement with `lyx loom decision add` is the parent's and lies outside the fixer's write surface.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: docs/overview.md:585
    summary: The batten entry still describes the removed Run-Shed window opening and its once-marker, and still says Run-Shed's side opens no VS Code, contradicting the new rows described a few lines above
    fix: In docs/overview.md replace the paragraph at lines 585-586 ("Run-Shed opens no VS Code window ..." and "On a run started with --window, Run-Shed opens that terminal window once per run ... a once-marker ...") so it no longer claims Run-Shed opens the window or keeps a once-marker; say that the Open-Window, Seed-IDE and Open-IDE rows own the window and VS Code, that a run resumed past Open-Window opens no window, and keep line 587 (window closing behaviour) as it is.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/battencli/cli.go:395
    summary: The --window envelope's terminal_window text says the window opens as soon as the task worktree is created, but the window now opens at the Open-Window row after Seed-Child, and only when the config or flag allows
    fix: In internal/battencli/cli.go:395 change the terminalWindow text to say the window opens through the launcher at the Open-Window row, and update any test or doc that asserts the old wording (grep for "as soon as the task worktree is created").
  - id: F4
    severity: LOW
    class: design
    location: internal/ideengine/spawn.go:574
    summary: The claim that SeedTask leaves the user's keybindings.json alone has no assertion; the batten integration test only checks that .vscode/ files appear
    fix: Extend an existing ideengine spawn test, without adding a top-level test, to call SeedTask and assert no file appears at the KeybindingsPath seam's path, while Spawn on the same fixture still seeds it.
---

### [MEDIUM] Row placement departs from the board entry

**Location:** contracts/recipes/batten-recipe.yaml:43
**Issue:** Scope item 2 of the board entry puts the rows "right after Worktree-Create, so the window still opens as soon as the pair exists".
The diff puts them between `Seed-Child` and `Run-Shed`, and the record has no `Added after Discussion` entry that licenses it.
The writer's reason is sound on its face: scope item 3 needs a task's `run_settings` to set the flags, and `Seed-Child` is where they reach the pair's local layer (`internal/battenshed/seamchild.go:144`).
The entry therefore conflicts with itself, which only the parent can settle.
The change description names it a design decision, not an unrecorded parent correction, but it has the same effect and needs the same handling.

### [MEDIUM] Stale batten paragraph in docs/overview.md

**Location:** docs/overview.md:585
**Issue:** The lines after the new row description still say that `Run-Shed` opens the terminal window once per run, after the child's first successful spawn, under a once-marker reset at a fresh batten seed.
That code is removed in this diff, so the doc contradicts the code and the new text at line 572.
Line 585 ("`Run-Shed` opens no VS Code window") is also now misleading beside the `Open-IDE` row.

### [LOW] Envelope text names the old window timing

**Location:** internal/battencli/cli.go:395
**Issue:** The `terminal_window` value for a launcher that exists reads "opens through <launcher> as soon as the task worktree is created".
The help text of the same command was updated to say the window opens once the run is seeded; this string was missed.

### [LOW] SeedTask's keybindings promise is unpinned

**Location:** internal/ideengine/spawn.go:574
**Issue:** The board entry requires the user-level key bindings to stay with `lyx ide`, and `SeedTask` is the only thing keeping `Seed-IDE` from touching them.
No test asserts it, so a later change that routes `SeedTask` through `seedKeybindings` would pass.
The `ideengine` test binary already points `KeybindingsPath` at a temporary directory, so the assertion needs no new fixture.

## Rejected

- Doubt over `PATTERN-config-strictness`: `battenshed` loads through `LoadOrTemplate` only and is added to the degrading set in the same diff; no violation.
- Doubt over the test budget: `shedrecipe` has 103 top-level tests once `TestMain` is excluded, equal to its row, and `shedadapters` was raised by one for its one new test.
- Doubt over duplicate tests: `switched_test.go` pins the producer's verdicts, `entries_switched_test.go` pins seam wiring and nil refusals, and the conformance table only registers the entries; no overlap found.
- Doubt over `--open-terminal`: `switchedActions` returns true for `Open-Window` before reading the file, so the override is per run and leaves the other two flags alone; a flag-read failure is returned as a row error and an action failure only warns.
- Leftover references to `OpenTerminalOnce`, `InnerRunDeps.OpenTerminal` and the once-marker: none remain outside the stale doc paragraph in F2.
