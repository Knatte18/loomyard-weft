```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it, or none**

1. `findWindowLocked` now filters on `@lym_hidden`, inverting the old `@lym_own` skip. `TestOpenWindow` in `window_test.go` pins it, in five rows:
   - A live hidden window is returned as existing (`ALiveWindowOfThatNameIsReportedAndNothingStarts`).
   - A live unhidden window is neither returned nor killed, and a new window opens (`ALiveUnhiddenWindowOfThatNameIsNeitherReturnedNorKilledAndANewWindowOpens`).
   - A dead unhidden window stays, with no `kill-pane`, and a new window opens (`ADeadUnhiddenWindowOfThatNameStaysAndANewWindowOpens`).
   - A dead hidden window is killed and replaced (`ADeadWindowOfThatNameIsKilledAndReplaced`).
   - A listing row with the flag `0` for the old `@lym_own` window reads as unhidden. That case is covered only through the unhidden rows.
   - If the filter reverted to `!= "1"` skipping, the first, second and fourth rows would fail.

2. `windowListFormat` now reads `@lym_hidden`. No test pins the format string. The fake answers by verb, not by format, and the integration test checks `@lym_hidden` through a separate `display-message`, not the listing. A format that wrongly read `@lym_own` would pass every fake-based row. That is a real gap, but it is covered at tier 0 only by the scripted rows, which cannot see the format. `window_integration_test.go:85` (from the grep hit) checks that the opened window carries `@lym_hidden`. It does not check that `OpenWindow` reuses the window on a second call against real tmux. I did not read the rest of that file, so I can't confirm the second-call reuse is absent.
   - Finding LOW, `design`, `internal/reedengine/window_test.go` with `window_integration_test.go:85`: no row proves a second real-tmux `OpenWindow` of the same name returns `Existing` through the new format.

3. `logBootWindowLocked` and its Info line, the Debug fallback and the non-fatal failure: none.
   - The grep for `bootWindowFormat`, `first window` and `opened the session's first window` finds no test.
   - The only `display-message` scripts in the boot tests are the identity reads in `tmuxconf_test.go:105-118` and `attach_test.go:134`. They answer by verb and do not exercise this call.
   - The new call runs on every fresh boot, so existing boot tests incidentally exercise the unscripted or wrong-shaped answer path (the Debug branch). That is the non-fatal property, but nothing asserts it.
   - Finding MEDIUM, `design`, `internal/reedengine/lifecycle.go:567-575`: the new behaviour has no test row for the Info line (id, pane, name parsed from three fields). It also has none for the failed or malformed read leaving the boot successful. A test would script `display-message` with `@0 %1 bash` and assert the logged fields, and with an error or `"x"` assert `ensureServerAndSessionLocked` still returns `booted=true`.

4. The `name` field added to the `opened own window` log at `spawn.go:328`: none. The grep finds no test asserting that log line. This is the log text only.
   - Finding LOW, `design`, `internal/reedengine/spawn.go:328`: the added `name` log field is untested.

5. The `name` field added to the `moved a strand pane out of the shell window` log at `ownwindow.go:439`: none. This is the same gap and the same kind of finding.
   - Finding LOW, `design`, `internal/reedengine/ownwindow.go:439`: the added `name` log field is untested.

6. `doc.go` and the doc comments are prose and exempt.

Findings 3 to 5 are all about log output. The repo may have no log-capture harness, so I rate 4 and 5 LOW. Finding 3 is MEDIUM because it adds a new tmux call on the boot path of every fresh session.

**Tool calls**
1. A `grep` over `*_test.go` in `internal/reedengine` for `display-message`, `first window`, the two log messages and `bootWindowFormat`. My context held `window_test.go` but no other test file, so I needed it to see whether any test covers the boot-window log or the added log fields.
