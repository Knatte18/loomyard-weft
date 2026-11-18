---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/settings.go:339
    summary: Both Python hooks invoke PATH's lym instead of resolving the executable through selfexec.
    fix: Resolve the executable through selfexec.LymBinary, or selfexec.ResolveLymBinary over an injectable lookup, during launch preparation. Pass the resolved path into the Claude and Codex Python hook renderers and shell-quote it. Update manifest command proofs and hook tests to use the same resolved path.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttlecli/hookpython.go:85
    summary: The hook selects hub config when launch mode selects standalone config for an existing Bolt directory without its _lym directory.
    fix: After ResolveWithin succeeds, use preflight.BoardLymPresent on the resolved location before selecting the hub stack. When it is false, retain the standalone-state fallback. Add a regression row with an existing Bolt directory, absent board-level _lym, and deny_python false in the target's standalone state; require exit 0 and verify subsequent toggles.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/hookpython_test.go:67
    summary: The hook verb's fail-closed response to a configuration-load error has no regression coverage.
    fix: Extend TestHookPythonAllowedVerb_FollowsTheHubsDenyPython with a malformed shuttle configuration row. Invoke the real command and assert exit 1 and a JSON error, exercising the LoadConfig failure branch rather than a directory-resolution failure or a stubbed lym exit.
---

### [BLOCKING] Resolve the hook executable through selfexec

**Location:** internal/shuttleengine/claudeengine/settings.go:339

**Issue:** The new Claude command invokes bare `lym`; Codex does the same in `codexengine/guards.go:38`.
`PATTERN-child-processes` explicitly requires hooks to resolve that executable through `selfexec.LymBinary`.
Neither renderer does so, and no commit supplies a `Pattern-Departure:` trailer.
A launch from a newer binary can therefore ask an older PATH binary that lacks the hidden verb.
The failed command then denies Python even when the launch's configuration allows it.
The repository constraint makes this finding BLOCKING.

### [MEDIUM] Match the launch's hub-presence criterion

**Location:** internal/shuttlecli/hookpython.go:85

**Issue:** The hook returns the hub stack whenever `fabricengine.ConfigDirs` finds a Bolt directory.
Launch mode instead uses `preflight.BoardLymPresent`, which checks the board-level `_lym` directory.
`preflight.ResolveMode` explicitly supports standalone fallback when that directory is absent.
The standalone launch reads its derived state directory in `internal/burlercli/wiring.go:184`.
With an existing `_bolt` but no `_bolt/_lym`, the hook ignores that state and loads the hub template.
Its default denies Python, even when the standalone state allows it.
This also prevents subsequent changes in the standalone state from reaching the hook.

### [MEDIUM] Cover configuration failures at the verb boundary

**Location:** internal/shuttlecli/hookpython_test.go:67

**Issue:** The hub table covers valid toggles and a directory-resolution failure.
The standalone table covers valid toggles only.
Neither reaches the new `LoadConfig` error branch in `hookpython.go:69–71`.
Loader tests establish loader errors, and hook-shell tests establish behavior for a stub's exit status.
Neither pins the new verb's conversion of a loading failure into exit 1 and a JSON error.
Dropping that error check would interpret the returned zero configuration as allowing Python.

## Review evidence

The review covers `71d6ee1a044d76998b1a3ff37a5eaec9113b83d5..HEAD` against the amended decision record.
All four required lenses returned reports, saved beside this file.
The scope lens found no unrequested changes.
The orch wiring already loads the hub layers, and the added launch test exercises that wiring.
Both engines install the Python row unconditionally and select its notice using the launch value.
The focused test-economy candidates retain assertions beyond their covering tests; neither is flagged.

The installed `lym` rejects `shuttle hook-python-allowed` as an unknown subcommand.
That runtime probe therefore did not execute the branch implementation.
No build or test pass is claimed by this review.
