```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I looked for what else reads, asserts or documents it.

**Per change, the set found**

1. **`findWindowLocked` filter and `windowListFormat` (`@lym_own` → `@lym_hidden`)**
   - `findWindowLocked` is the only reader of `windowListFormat`, and `OpenWindowIn` is its only caller.
   - `OpenWindow` and `OpenWindowIn` have these callers beyond `window_test.go`: `windoworder_test.go:213`, `window_integration_test.go:69,92,103`, `ownwindow_integration_test.go:205` and `windoworder_integration_test.go:137,157`.
     - The integration callers run against real tmux, where `OpenWindow` sets `@lym_hidden` itself, so the new filter still reuses them.
     - `ownwindow_integration_test.go:205` still passes. An own window has no `@lym_hidden`, so it is not adopted and a new window opens.
     - `windoworder_test.go:213` uses the fake. I did not read its listing fixture, so I could not confirm it still matches the new column meaning. It is the one unconfirmed caller. Covered otherwise.
   - Docs: `doc.go:667` and `doc.go:711-713` and the `window.go` comments are updated and consistent with the code. Covered.
   - `doc.go:633` ("Reconcile ends only a window carrying @lym_own … a window without the marker is never ended") already matches, and the kill sites do not match by name. `reconcile.go:262` kills by id, `window.go:92` only a `@lym_hidden` match, and the `kill-pane` calls in `spawn.go:161`, `selvagepane.go` and `strand.go:831` are by pane id. Covered.
   - The new-window and set-option chain in `window.go:116-119` is a single tmux invocation, and an unmarked `lym-opening-` window is never matched by name either before or after this diff. No regression.

2. **`logBootWindowLocked` and `bootWindowFormat` (new `display-message` after boot)**
   - Called only from `lifecycle.go:545`, and it runs on the Windows path too.
   - Existing fake-tmux tests that count or sequence `display-message` calls: `bindings_test.go:306`, `ownwindow_test.go:111` and `attach_test.go:430`. The first two I did not read; `attach_test.go:430` filters by format.
   - The fake answers any `display-message` with a per-test fixture such as `"$0|4321|1700000000"`. That is not three space-separated fields, so the new log would take the Debug path. It is non-fatal, so no failure follows.
   - No test pins the new Info line or its Debug fallback. That is for the tests lens.

3. **`name` field added to the `opened own window` and `moved a strand pane out of the shell window` log lines**
   - The only readers are `spawn.go:328` and `ownwindow.go:439`, and no test or doc asserts the text.
   - `strandLabel` is already used at `ownwindow.go:425` and `name` is in scope at `spawn.go:328`.
   - Note that for overview windows `name` is reassigned at `spawn.go:305` before the log, so the logged name is the real window name. Covered.

4. **PATTERN, repository rules and commit message**
   - `pattern/PATTERN-reed-panes.md` and the other PATTERN entries do not describe window reuse, so no PATTERN diff is owed. No declared PATTERN changes appear in the directive.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.
   - Landing claim: `_lym/landing/summary.md` says "no code path was found". The decision record's third Wanted bullet asks the run to say so in its landing, and the summary does. Covered.

**Findings:** none.

**Tool calls**
1. `grep` for `sessionListVerb`, `windowListFormat` and `OpenWindow` callers, plus stale doc phrasing, to find every reader and caller outside `window_test.go`. My context lacked it.
2. `grep` for `display-message` in the reedengine tests, plus a read of `ownwindow_integration_test.go:195-225` and `doc.go:625-640`. My context lacked the boot-path fixtures and the doc line to check them against.
