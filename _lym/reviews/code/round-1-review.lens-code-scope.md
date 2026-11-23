```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the board entry, since the answer key is a decision record alone.

**Decisions** (the board entry's `## Goal`; the record holds no `Added after Discussion` entries):
- **G1.** After a failed click, the bar shows why ("the bar … shows why it did not").
- **G2.** After a failed click, a log shows why ("or a log shows why"). The entry also notes that no trace log of a `slack toggle` run was written.
- **G3.** "Expected: a click flips the button, or the bar or a log shows why it did not." This is a disjunction, and G1 and G2 satisfy it. The entry does not ask the diff to find or fix why the operator's particular click failed.

**Changes → decisions:**
- **C1. A pre-run refusal is reported on the clicked bar** → G1.
  - `internal/slackcli/cli.go:194-200`: `preRun` calls `reportToggle` when the verb is `toggle`.
  - `internal/slackcli/wiring.go:56`: `showToggle` moves into `productionCLI`, and `wireProduction` drops its assignment.
  - `internal/slackcli/cli.go:102`: new field comment.
- **C2. Every failed toggle is logged at Warn** → G2.
  - `internal/slackcli/toggle.go:156`: the `logger.Warn` in `reportToggle`.
- **C3. The toggle's message is held on the bar for ten seconds** → G1.
  - `internal/reedengine/slacktoggle.go:11-13,24`.
  - The old 750 ms display-time left the bar unreadable, so a refusal shown that briefly does not "show why".
  - Only failures pass a non-empty message, because `reportToggle` sends `""` on success. The hold therefore adds nothing outside G1.
- **C4. `reportToggle` and the flag and verb constants** → forced by C1.
  - `internal/slackcli/toggle.go:22-28,110,140-163`.
  - `preRun` has to read the toggle's flags and recognise the verb outside `toggleCmd`'s closure.
- **C5. Documentation** → forced by C1 to C3.
  - `internal/slackcli/doc.go:42-43`, the long help at `internal/slackcli/toggle.go:127-130`, and `docs/overview.md:954`.
  - These describe the verb's behaviour, which the change alters.
- **C6. Tests** → forced by C1 and C3.
  - `internal/slackcli/toggle_test.go:173-186` (new subtest).
  - `internal/reedengine/slacktoggle_test.go:36-38,66-69` (expectations now include `-d 10000`).

**Decisions → changes:**
- G1 → C1, C3, with C4, C5 and C6 forced by them.
- G2 → C2, with C5 forced by it.
- G3 → C1, C2 and C3 together. The diff does not claim to fix the unknown cause of the original failure, and the entry's "or" does not require it.

**Findings:** none. Every hunk traces to a goal point or is an edit a traced change forces, and every goal point has at least one change that builds it.

Tool calls: none.
