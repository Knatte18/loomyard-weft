---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/overview.go:395
    summary: atOrBelow follows the parent chain with no cycle guard, so a cyclic chain hangs a beam window's frame
    fix: In atOrBelow, track the slugs already visited in a map and return false with an error naming the repeated slug when a slug recurs, so orchsAtOrBelow leaves that orch out and logs it at Debug like any unreadable chain. Add a TestOrchsAtOrBelow row whose parentOf maps a to b and b to a, with the window's worktree outside the cycle, and assert the call returns and leaves a and b out.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:410
    summary: "the orch module row still calls the runs strand's window the session's overview window and lists the windows as overview, agents, shell"
    fix: "In the orch row of docs/overview.md, say that start ensures the worktree's runs strand, which runs lyx batten overview as the beam's window named beam: prime or beam: <slug>, and change the phrase about the session's windows running overview, agents, shell so that it names the beam window first."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/beamcli/cli.go:87
    summary: the dobbyStatus wiring that prints run cells in lyx beam status has no test, since the existing status test has no runs
    fix: Add a row or assertion to TestStatus_PrintsThePrimesDobbyAndEachLocalBeams in internal/beamcli/status_integration_test.go with one run in the dobby view, and assert the entry's Lines hold a run line with its state and stage cells.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/battencli/overview.go:326
    summary: the orchs field comment and the orchLines comment say the window's own orch comes first, but windowOrchPaths keeps HubOrchs order, where a nested beam can precede it
    fix: Either drop "its own first" from the comments at overview.go:326 and :409, or make windowOrchPaths move the window's own orch to the front of the shown list and cover it in TestOrchsAtOrBelow.
---

### [MEDIUM] atOrBelow has no cycle guard

**Location:** internal/battencli/overview.go:395
**Issue:** `atOrBelow` loops until the slug is empty or the prime, and has no visited set. A self-parent or two beams naming each other as parent, in a malformed origin record or board parent, never ends for a beam window whose worktree is outside the cycle. `DobbyParents.Of` answers from its cache, so the pane's frame function spins and the pane stops redrawing. The prime's window returns early and is unaffected. The focus directive asks that the scope terminate on a cyclic chain; a missing chain does terminate and is covered by the `broken` row of `TestOrchsAtOrBelow`.

### [LOW] Orch row of docs/overview.md still names the overview window

**Location:** docs/overview.md:410
**Issue:** The scope lists `docs/overview.md` among the docs to update. The diff changed the `lyx beam status` and `lyx batten overview` entries but left the orch module row calling the window the session's overview window, with windows "overview, agents, shell".

### [LOW] lyx beam status run cells have no test

**Location:** internal/beamcli/cli.go:87
**Issue:** `dobbyStatus` now builds `NewDobbyRunCells` and passes the cells to `DobbyLines`. The acceptance criterion that `lyx beam status` prints the pane's dobby lines is unpinned: the existing status test uses dobbies with no runs and passes with the cells wiring removed.

### [LOW] Orch order claim does not hold

**Location:** internal/battencli/overview.go:326
**Issue:** `windowOrchPaths` returns the shown orchs in `HubOrchs` order (prime first, then worktree list order, `internal/orchcli/live.go:28`). In a beam's window a nested beam can precede the beam's own orch, against the comments saying its own comes first. Only line order is affected.

## Rejected

- Untested wiring of `runsWindowTitle` from the worktree name and beam flag at `internal/orchcli/cli.go:194` (code-tests, NIT): the helper `runsWindowTitleOf` and the strand spec's title are both pinned, and the remaining line is a one-line assignment.
- `DobbyLines` drawing blank cells for a slug missing from the cells map (code-correctness, noted as not wrong): both callers pass `cells.Of(view.Runs, ...)`, so no slug is missing.
- No test that a run is drawn only once (code-tests, not raised as a gap): the run list is gone from the code, so only reintroducing a list could break it.
