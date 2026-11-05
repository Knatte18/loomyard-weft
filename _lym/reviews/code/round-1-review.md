---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shuttleengine/codexengine/capabilities.go:267
    summary: "The Goal asks for the idle wake through codex queue from the daemon, but the diff runs it in the hook-wait waiter process, and no Added after Discussion entry records the change."
    fix: Leave the code in place and defer this finding to the run's parent, since its only sound change is recording or refusing the deviation with lym loom decision add, which lies outside the fixer's write surface. Do not revert WakeIdle, IdleWaker or wakeSession.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelcli/hookwait.go:18
    summary: "The hookWaitVerb and hookReadVerb comments still say the Claude engine's hooks spell the verb, though the Codex engine now spells both in its hook commands."
    fix: In internal/creelcli/hookwait.go:18 reword the hookWaitVerb comment, and in internal/creelcli/hookread.go:19 the hookReadVerb comment, to say the verb is the one a strand's engine hooks spell, naming no engine.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/prepare_test.go:299
    summary: "No test pins that the waiter's LYM_CODEX_BINARY comes from codex.yaml's binary setting; TestPrepare_CreelHooksInAHub expects the default codex, so a hard-coded codex in Prepare would pass."
    fix: In TestPrepare_CreelHooksInAHub set a non-default Binary in the engine's codex.yaml and expect it as the bin of the creelDelivery the test builds, so the settings it looks for in the launch line carry that binary.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/claudeengine/settings.go:123
    summary: "Two indirections stay in claudeengine after the move, the strandFlagName wrapper around StrandFlagNameShell and the creelTrustSentence constant aliasing CreelTrustSentence, which the code-quality rule on internal shims says to replace by changing the call sites."
    fix: In internal/shuttleengine/claudeengine/settings.go delete strandFlagName and call shuttleengine.StrandFlagNameShell() at its two call sites, and delete the creelTrustSentence constant and use shuttleengine.CreelTrustSentence at its uses in settings.go, settings_test.go and prepare_test.go; the diff already does this for CreelFlagDir.
---

### [MEDIUM] Idle wake runs in the waiter, not in the daemon

**Location:** internal/shuttleengine/codexengine/capabilities.go:267
**Issue:** The Goal's Wanted list names "The idle wake through `codex queue` from the daemon" and asks for a unit test on "the daemon's wake call".
The diff runs `codex queue` inside `WakeIdle`, called from `hookWait`, the waiter hook process the daemon rings.
The change description records this as a design decision and argues it meets the same need, but the decision record carries no `Added after Discussion` entry that amends the Goal, so the deviation is unrecorded.
The code itself reads correct, and the lens found no defect in it; the open question is whether the parent accepts the waiter as the place of the wake.

### [LOW] Stale verb comments name only the Claude engine

**Location:** internal/creelcli/hookwait.go:18
**Issue:** The `hookWaitVerb` comment says the verb is the one "the Claude engine's waiter hook spells", and `hookReadVerb` at `internal/creelcli/hookread.go:19` says the same of "the Claude engine's hooks".
The Codex engine's `creelWaitCommand` and `creelMailCommand` now spell both verbs, so the comments are false.
The diff reworded the neighbouring `hookWaitCmd` and `hookWait` comments and missed these two.

### [LOW] Waiter's configured Codex binary is not pinned

**Location:** internal/shuttleengine/codexengine/prepare_test.go:299
**Issue:** `Prepare` builds the waiter's `bin` from `codexBinary(codex)`, which reads `codex.yaml`'s `binary`.
`TestPrepare_CreelHooksInAHub` builds its expected settings with `bin: "codex"`, the default, so a `Prepare` that hard-coded `codex` would still pass.
`TestWakeIdle` always sets `LYM_CODEX_BINARY`, so the `codexBinary(Config{})` fallback in `WakeIdle` is also unexercised; that fallback is a trivial default and is not raised on its own.

### [LOW] Leftover indirections after the move into shuttleengine

**Location:** internal/shuttleengine/claudeengine/settings.go:123
**Issue:** `strandFlagName` returns `shuttleengine.StrandFlagNameShell()` and nothing else, and `creelTrustSentence` is a constant equal to `shuttleengine.CreelTrustSentence`.
Both exist only so the old call sites keep compiling, while the same diff updated every call site of `CreelFlagDir` to the new home.
The code-quality rule against shims for internal code asks for the call sites to change instead.

## Rejected

- Code-tests F2, no test of `promptText` with an empty deny notice: the empty-notice branch is the unchanged `if notice != ""` guard, the diff only changes what heads the text, and the Prepare tests pin the head and the passages after it for the installed guard rows.
- No finding on Claude's hook behaviour: the code-correctness lens ran the moved fragments and found their bodies byte-identical to the originals, and `wakeSession` keeps the old stderr write and wake code for Claude.
- No finding on the Codex hooks not changing directory first: the change description records it as a decision, and Codex runs its hooks in the session's working directory.
