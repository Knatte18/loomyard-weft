---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/reedengine/panebin.go:319
    summary: the directory change of the pinned click line and the confirmed line sits outside the output discard, so a gone run directory writes cd's error to stderr and opens view mode
    fix: In composeButtonCommand (panebin.go:319) and composeConfirmedButtonLine (panebin.go:339) wrap the directory-change statement in sh.DiscardOutput too, i.e. sh.Chain(sh.DiscardOutput(sh.ChangeDirToWord(...)), sh.DiscardOutput(verb), sh.ExitSuccess()) and likewise with sh.ChangeDir(dir) for the confirmed line. Add a row to the panebin_test.go click-line case, for both dialects, that asserts the cd statement carries the discard, and a real-tmux row in the bindings integration test that clicks a button whose run directory was removed and asserts the pane is not in view mode.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/panebin_test.go:340
    summary: the pinned click line has no pwsh row, so the pwsh forms of the discard, the zero exit and the directory change are unpinned
    fix: Add a pwsh row beside the POSIX one in the composeButtonCommand test in panebin_test.go, asserting the exact line (Set-Location -LiteralPath, the verb followed by *> $null, then exit 0), and a direct assertion of ExitSuccess in shell_test.go for both dialects.
---

### [BLOCKING] Run directory change is outside the output discard

**Location:** internal/reedengine/panebin.go:319 and internal/reedengine/panebin.go:339
**Issue:** D7 requires that no click opens a view-mode pane, a gone run directory included, and says the discarded output with the forced zero status keep view mode shut whatever happens.
Both pinned lines are `Chain(cd, DiscardOutput(verb), exit 0)`, and Chain joins with `; `, so only the verb is redirected.
When the run directory is gone, `cd` (pwsh `Set-Location`) writes an error to stderr and tmux collects it as run-shell output.
I probed tmux 3.6: `run-shell -b 'cd /nonexistent_x; exit 0'` leaves the target pane in view-mode, as do plain stdout and stderr output with `exit 0`.
The verb's own "directory is gone" message (buttonclick.go:96) therefore arrives after view mode has already opened, and the D7 acceptance line "no click, refused or failed included, opens a view-mode pane" fails on this path.

### [LOW] Pinned click line has no pwsh test row

**Location:** internal/reedengine/panebin_test.go:340
**Issue:** The click-line test pins the POSIX form only; the pwsh forms of the discard (`*> $null`), the directory change (`Set-Location -LiteralPath`) and the forced `exit 0` have no row, and `ExitSuccess` is not asserted on its own in shell_test.go.
The confirmed line is pinned for both dialects, so only the click line is uncovered.

## Rejected

- Code-affected-set F1 (`toolfloor/doc.go` omits `run-shell -b -c`): the diff does not use `run-shell -c`. I probed tmux 3.6: `-c '#{@d}'` is not format-expanded, so the implementation changes directory inside the line instead, and the floor list is accurate. The difference from D7's text is sound and no PATTERN entry forbids it.
- Code-tests F1 (no test for a panic or a failing `display-message` on exit 0): a panic cannot be tested through the verb, since the forced `exit 0` in the line is the contract, and F1 above covers that line. The failing-message path has `FailedMessageIsLoggedAtDebugOnly`.
- Code-correctness minor note (`validateButtonFlags` sets a non-zero exit): reachable only by a direct call with missing flags, not one of the paths D7 lists, and the click line forces zero.
- Code-tests F3 (`ParentOf` returning "" unpinned): the lens did not read the test bodies and names no missing behavior with certainty.
- Code-tests F4 (`@strand` and `@lym_strand_name` set together): `paneoptions.go` is the sole setter and `spawn_test.go` rows pin the option, including the empty-name unset.
- Code-tests F5 (possible integration overlap): the lens names no redundant test with certainty, and the two real-tmux tests drive different seams (the verb versus the pinned binding).
