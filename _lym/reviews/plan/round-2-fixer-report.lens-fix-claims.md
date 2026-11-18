```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every claim the fix added or rewrote against the code at HEAD.

**LOW.** Card 5 Wiring, the sentence "every existing test that builds a `battenCLI` literal and calls `wire` reaches the real pick".
- **Summary:** the claim is too broad. Only tests that drive `CreateWorktree` past `top.Add`, or into the `complete` branch, reach the pick.
- **Evidence:**
  - `internal/battencli/wire_seams_test.go:259` and `:325` call `wire` and never call `CreateWorktree`.
  - The rows of `TestWire_CreateWorktreeClaimsTheBoardEntry` fail on the absent fabric config before `top.Add` (doc comment at `wire_seams_test.go:709`).
- **Suggested wording:** "every existing test that calls `wire` and then drives `CreateWorktree` past the add reaches the real pick".

**LOW.** Card 14 step 3, the new sentence "With `startPath` set, a died or timeout end goes the start path's way".
- **Summary:** a timeout end only arises in step 4 (a pane that never reaches ready), so stating it in step 3 attributes timeout to steps 2 to 3. This is a placement slip; the rule itself holds.
- **Evidence:** card 14 step 4: "A pane that never reaches ready ends the run … died, or timeout when the run's own deadline passes first."

These claims hold:
- **`finalize` is not idempotent, and `Wait` would finalize again:**
  - `wait.go:1978` has no guard against a second call.
  - `Wait` (`wait.go:268`) has no early return on a terminal record.
- **`abandonStartup` finalizes once, removes the strand and wraps `ErrNotStarted`:** `wait.go:847-869`.
- **`start` still judges its retry:** `run.go:630` (`startRetryable` needs `startupStrandRemoved`, which `abandonStartup` sets).
- **The done branch:** "leaves the strand … `Wait` ends the run done once" holds. `Wait`'s liveness path answers `OutcomeDone` when every output file exists, at `wait.go:1486`, `wait.go:1495` and `wait.go:1737` (`classifyDeadlineExpiry`).
- **The new card 14 test row "`Start` return no run and an error wrapping `ErrNotStarted`" with no start retries:** `run.go:630-631` and `StartGated` at `run.go:539-544`.
- **The owner-seam tests override `owner` after `wire`:** `wire_seams_test.go:795`.
- **The doc.go paragraph openings quoted in cards 14 and 16:** `doc.go:161`, `doc.go:190`, `doc.go:570` and `doc.go:574`.

Tool calls:
- One grep of `wait.go` for how `classifyStartupWindow` and `checkLivenessTick` end a dead strand. My context did not show how `Wait` ends a strand with no process when every output file exists.
