---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-lym-files.md:25
    summary: The rule still says only a hub-wide module's machine override sits under the board dir, though codex now reads one too.
    fix: In pattern/PATTERN-lym-files.md line 25, change "A hub-wide module's machine override sits under" to "A module's machine override sits under", keeping the rest of the line.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:513
    summary: The overview still says only a hub-wide module's machine keys take a machine override, though codex's sandbox_mode is now a per-worktree machine key.
    fix: In docs/overview.md line 513, reword "A hub-wide module's machine keys, such as gate's `slots` and `tier_parallel`, also take a machine override." to "A module's machine keys, such as gate's `slots` and `tier_parallel` and codex's `sandbox_mode`, also take a machine override."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/open_test.go:81
    summary: No test sends danger-full-access through OpenLaunch, so a hardcoded workspace-write there would pass every test.
    fix: Add a row to TestOpenLaunch in open_test.go whose worktree layer writes sandbox_mode danger-full-access, and expect the codexCommand line built with sandboxMode set to sandboxFullAccess.
---

## Verdict

The diff makes the change the board entry asks for: a `sandbox_mode` key, `workspace-write` by default, `danger-full-access` allowed, settable per machine.
The entry's preflight is optional ("either or both"), and the commit leaves it out.
The launch line under `danger-full-access` keeps `-a never` and drops the network and writable-root settings.
An empty or unknown value refuses the load with a way forward, and does not fall back.
The machine layer stacks between worktree and local only for a module with a machine key and both `Anchor` and `Board`.
`codex` is the only per-worktree module with such a key, so no other module's stack changes.
A local override of `sandbox_mode` is refused.
The config-layer change and the `configreg` test rewrite are needed to make the key machine-settable, so the entry licenses them.
The removed `configreg` test enforced a rule the diff reverses; the per-module `MachineKeys` check remains.
No PATTERN changes are declared, and the commits need no `Pattern-Departure:` trailer.

### [LOW] Stale hub-wide wording in PATTERN-lym-files

**Location:** pattern/PATTERN-lym-files.md:25
**Issue:** The line describes the machine override as a hub-wide module's file only, which the layer change makes false.

### [LOW] Stale hub-wide wording in the overview

**Location:** docs/overview.md:513
**Issue:** The line says only a hub-wide module's machine keys take a machine override, though the same diff documents `sandbox_mode` at line 693.

### [LOW] OpenLaunch carries danger-full-access untested

**Location:** internal/shuttleengine/codexengine/open_test.go:81
**Issue:** Only a workspace-write opened session is asserted. An opened session on a machine without a working bubblewrap would still stall if `OpenLaunch` stopped passing the mode.

## Rejected

- Code-tests lens finding 2, no `TestDispatchLayers` row for an invalid `sandbox_mode` set through `--layer machine`: the refusal-and-restore path is generic code that the existing `gate` row pins, and the mode check itself is pinned by `TestPrepare_RefusesBeforeWriting/UnknownSandboxMode`. A codex row would duplicate that coverage.
