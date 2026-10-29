---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/ideengine/spawn_hub_integration_test.go:299
    summary: A prime Seed whose hub workspace regeneration fails has no test, though Seed returns that error
    fix: In checkPrimeWriteFailure, also call Seed(l, primeOf(t, h)) against the planted read-only workspace file and assert it returns an error containing "write workspace file" and leaves the planted bytes.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/fabricengine/hubworkspace_integration_test.go:345
    summary: workspaceWindowTitle's no-shortname fallback to "<name> workspace" has no test
    fix: Add a subtest of TestRegenerateHubWorkspace on a hub whose board records no shortname, asserting the hub file's window.title is "<RepoName> workspace" and a beam file's is "<slug> workspace".
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/vscode/config.go:18
    summary: Package doc Limitation still says BuildWorkspace splices the prime's settings verbatim into the hub file, which WorkspaceSettings now re-encodes first
    fix: Reword the Limitation so it says the hub workspace file's settings carry the prime's settings, keeping the relative-path consequence.
    origin: lens:code-affected-set
---

### [LOW] Prime Seed regeneration failure untested

**Location:** internal/ideengine/spawn_hub_integration_test.go:299
**Issue:** The decision record's D4 says `Seed` returns a regeneration error as `Spawn` does. `checkPrimeWriteFailure` drives `Spawn` only, so breaking `Seed`'s error return fails no test.

### [LOW] No-shortname workspace title untested

**Location:** internal/fabricengine/hubworkspace_integration_test.go:345
**Issue:** D2 names the `<name> workspace` fallback for a board with no shortname. Every new test runs with the `TST` shortname, so the fallback branch of `workspaceWindowTitle` is unpinned.

### [LOW] Stale "verbatim" claim in the vscode package doc

**Location:** internal/vscode/config.go:18
**Issue:** The doc was edited above this line for the workspace look, but the Limitation still describes the hub file's settings as spliced verbatim; the regeneration now passes them through `WorkspaceSettings`.

## Rejected

- Code-tests NIT on shared-seam coverage of the skipped keybindings outcome and the no-slug prime seed: both paths converge on code the Spawn and explicit-slug tests already pin, so a new row would duplicate coverage.
- Code-tests item 8 (unchanged regeneration): the existing idempotence assertion in `checkPrimeSplicesSettingsAndRegenerates` covers it and the output is key-sorted, so no gap.
- Float64 precision and HTML escaping in `WorkspaceSettings`: semantically equal JSON, same as `writeSettings`.

Holistic review found the diff matching the amended board entry and decision record: scope is licensed (docs/overview.md and the battenshed template comment describe the new seed behavior), no PATTERN entry is touched, and no stencil changes.
