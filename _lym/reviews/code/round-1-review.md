---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedengine/lifecycle.go:567
    summary: >-
      logBootWindowLocked, a new tmux call on every fresh boot, has no test for its Info line or its non-fatal failure path
    fix: >-
      Add a test in internal/reedengine, beside the existing boot tests that use the scripted fake tmux, that scripts display-message to answer "@0 %1 bash" and asserts the Info line "reed: opened the session's first window" carries window @0, pane %1 and name bash. Add a second row that scripts display-message to fail, and one that answers a single token, and assert ensureServerAndSessionLocked still returns booted=true with no error. Use a log capture the package already has, or the logger's test sink, and do not add a new harness if none exists; in that case pin only the non-fatal rows.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: internal/reedengine/doc.go:665
    summary: >-
      Package doc says every window reed opens is opened with a name, but the boot's first window is created by new-session without -n and starts named after its shell
    fix: >-
      In the window-order section of internal/reedengine/doc.go, reword the line "Every one of them is opened with a name, so a window named after its shell came from no reed step." so it states the truth for the boot window. For example, say that an own window, a break-out and a detached window are each opened with a name, and that the boot's first window starts named after its shell and is renamed shell by the shell-window step, so a window still named after its shell outside that boot came from no reed step.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/spawn.go:328
    summary: >-
      The name field added to the "opened own window" and "moved a strand pane out of the shell window" Info lines has no test
    fix: >-
      Pin the name field with a row in the existing test that drives each path (openOwnWindowLocked at spawn.go:328, moveShellWindowStrandsLocked at ownwindow.go:439) only if the package has a log capture to assert on. If it has none, record that the field is log-only and leave it; do not add a log-capture harness for this finding alone.
---

### [MEDIUM] Boot-window log has no test

**Location:** internal/reedengine/lifecycle.go:567
**Issue:** `logBootWindowLocked` adds a `display-message` round trip to every fresh boot and parses its answer into three fields.
The decision record's third Wanted bullet asks for a trace line naming every window reed opens, and this is the only line for the boot window.
No test pins that the line names the window id, pane id and name, or that a failed or malformed read leaves the boot successful.
Existing boot tests reach the Debug branch only by accident, because the fake's `display-message` fixtures are not three space-separated fields.
The code itself ran correctly on every input the correctness lens tried.

### [LOW] Package doc overstates that every opened window is named

**Location:** internal/reedengine/doc.go:665
**Issue:** The added line says every window reed opens is opened with a name.
The boot's first window comes from `new-session` in `serverSpawnArgv` (lifecycle.go:624), which passes no `-n`, so it starts named after its shell.
The change description says as much, and says the shell-window step renames it.
The doc line contradicts the code and the description it ships with.

### [LOW] Added log name fields are untested

**Location:** internal/reedengine/spawn.go:328
**Issue:** The `name` field on the own-window line (spawn.go:328) and on the break-out line (ownwindow.go:439) is new and has no assertion.
The values were checked by reading: the own-window line logs the post-overview-rename `name`, and the break-out line uses the same `strandLabel(s.Name)` as the `break-pane -n` argument.
The gap is log text only, so it is rated LOW.

## Focus answers

- `logBootWindowLocked` target `=<session>:` is the same target `serverPIDLocked` already uses with `display-message -p` on both platforms.
  A fresh session holds one window, so it names that window.
  A failed or unparsable read logs at Debug and returns on every boot path, since the call sits after the session is confirmed up and all failures return early.
  Not verified against psmux live.
- `findWindowLocked`: every window `OpenWindow` opens gets `@lym_hidden` in the same chained tmux invocation as `new-window`, not at creation.
  A failure partway through the chain leaves a window under the temporary `lym-opening-` name.
  That window is neither reused nor replaced, and it was equally unreachable by name before the diff, so the inversion adds no leak.
- First Wanted bullet: the diff finds no path that leaves a command-less window and says so in `_lym/landing/summary.md`, as the third bullet requires.
  Every `new-window` and `break-pane` in the package passes `-n`, which I confirmed by grep.
  The one unnamed window is the boot's, and its source is documented and now logged.
- Third Wanted bullet: the windows reed opens are the boot window, an own window, a break-out and a detached window.
  The only sites are `new-session` (lifecycle.go:624), `newWindowLocked` (spawn.go:253, reached from `openOwnWindowLocked` and `OpenWindowIn`) and `break-pane` (ownwindow.go:425).
  Each now logs Info with window id and name, and pane id except the detached-window line, which logs the window id, name and command, since a detached window is opened with a window-id-only format.
  The decision record asks for window id, pane id and name; the detached line carries no pane id, which I judge acceptable since the window id addresses it and the record's purpose, tracing a stray window's source, is met.
- Reed closes only windows it opened: reconcile ends windows only through `ownWindowsHoldingNoStrand`, which requires `Own`, and kills by id.
  `OpenWindow` kills only a hidden dead window's pane.
  The untracked-pane reap reads window 0's panes only.
  No path ends or adopts a window by name alone.
- `window_test.go`: the rows cover live and dead windows, hidden and unhidden, against the new filter.
  `ASecondCallWhileLiveStartsNothing` in `window_integration_test.go` pins reuse through the real listing format.

## Rejected

- Lens code-tests, LOW on `windowListFormat` having no pinned format: rejected, because `ASecondCallWhileLiveStartsNothing` in `window_integration_test.go` calls `OpenWindow` twice against real tmux and requires `Existing`, which fails if the format reads the wrong option.
- Lens code-correctness note that the hidden marker is not set at creation: rejected as a finding, because the window is unreachable by name until the rename and the old filter had the same exposure.
- The change description's claim that the diff adds no fix for a command-less window: the diff bears it out, and no finding is raised.
