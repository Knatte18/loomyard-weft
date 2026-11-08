---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/orchengine/prompt.go:9
    summary: doc comments still call the presence the hub's after it became the machine's
    fix: Reword "the hub's presence" to "the machine's presence" in internal/orchcli/paths.go:22, internal/orchengine/state.go:42, internal/orchengine/prompt.go:9, :28 and :31, internal/orchengine/doc.go:52 and internal/dobbyengine/waitreminder.go:28.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackcli/hook.go:140
    summary: a failed creel read in the prompt hook and a failed post in the reply hook leave no trace, because the turn hook discards all output
    fix: In promptHook (the creel Log failure at hook.go:140-143) and replyHook (the Post failure at hook.go:177-179), call logger.Warn with the strand and the error before returning it, so a lost reply or a missed switch back shows in lym's log.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/testmain_test.go:19
    summary: the TestMain guard that exits 2 on a positional argument has no test, and it is the second layer against the reedengine fork bomb
    fix: Add a test in internal/reedengine that re-executes os.Executable() with one positional argument (a helper-process pattern guarded by an environment variable) and asserts exit status 2 with no suite output, so a regression of the guard fails a test.
---

### [LOW] Doc comments still call the presence the hub's

**Location:** internal/orchengine/prompt.go:9 (also :28, :31), internal/orchengine/state.go:42, internal/orchengine/doc.go:52, internal/orchcli/paths.go:22, internal/dobbyengine/waitreminder.go:28
**Issue:** The diff moves presence to the machine (`PresencePath` in `hubgeom.SlackMachine`), but these comments still describe it as the hub's record, which contradicts the new `presence.go` doc.

### [LOW] Hook failures leave no trace

**Location:** internal/slackcli/hook.go:140
**Issue:** `turnHookStep` sends the hook's output and errors to /dev/null, and neither `promptHook` nor `replyHook` logs a failure.
A creel `Log` error leaves presence at slack, and a `Post` error drops the turn's reply, in both cases with nothing for the operator to find.
The decision record asks that failure paths be handled; returning the error alone does not do that when the caller discards it.

### [LOW] The TestMain guard is untested

**Location:** internal/reedengine/testmain_test.go:19
**Issue:** The record's amendment asks for a guard that exits at once on non-test arguments, and a test that a pinned status job in a test session never launches the test binary.
The second exists (`TestSlackLines_RealTmuxBootByATestBinaryPinsNeither`, `TestBinaryExecutable` row).
The guard itself is the fallback if `slackLineExecutable` is ever bypassed, and nothing fails if it is removed.

## Rejected

- Production `reedengine` imports `testing` for `testing.Testing()` (affected-set, handler): no PATTERN entry forbids it, and other production packages (`orchcli`, `beamcli`, `gitkit`) already do the same.
- Test-binary loop in the other pinned commands (focus item 1): `follow`, `button` and `switch` lines run only on a click, never at a redraw, and the hook command is the literal `lym slack hook` resolved through PATH, so none can run a test binary; the state job and toggle click are refused by `slackLineExecutable`.
- code-tests: the toggle and state line content has no test (MEDIUM): `panebin_test.go:450-505` asserts the exact toggle and state lines on posix and pwsh, including `##` doubling and the `slack bar` text.
- code-tests: `TurnHookCommand` on the orch spec is not asserted: `orchcli/start_test.go:491` asserts it equals `lym slack hook`.
- code-tests: the beams-read-failure presence notice has no row: `relay_test.go:958` (`FailedBeamsReadMailsTheHubOrchAlone`) covers it.
- code-tests: the `diskCheck` no-watch path has no notice row: `relay_test.go:1120` runs a relay with a failing watch; the finding was raised without reading it, so I did not keep it.
- code-tests: the inject window is unpinned (NIT): `hook_test.go:146` asserts the `Since` filter of `now - 2 minutes` passed to the journal.
- code-correctness: a permanently failing dial lights `slack` (LOW): the summary's design notes and `Connected`'s doc state that a held lock counts as connected, and the decision asks for a toggle lit while the relay is connected, not while the dial succeeds.
- Presence per machine and the auto-switches (focus items 2 to 4), the Stop hook (item 5), the toggles (item 6) and the `<machine>::<sender>` names (item 7): traced by the correctness fork and by me, and no defect was found; each is covered by tests the code-tests fork listed.
- Scope against the record (focus item 8): `role-slack.md` keeps one plain line on sending through creel, as the record asks; `orch-role.md` and `docs/overview.md` changes follow from that and from the new behaviour.
- Summary and commit claims (focus item 9): the claims I checked against the diff hold, and the `4aa372a7`-era commit messages match the code; no `Pattern-Departure:` trailer is present or needed.
