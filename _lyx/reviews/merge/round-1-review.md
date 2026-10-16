---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/creelcli/smoke_helpers_test.go:86
    summary: addStrand comment states that the default anchor splits the strand into the selvage shell window (issue 673), but the merged parent made own-window the reed add default, so the comment is false and the --anchor own-window argument is redundant
    fix: In addStrand in internal/creelcli/smoke_helpers_test.go, drop the "--anchor", "own-window" pair from the reedcli.RunCLIIn argument list and rewrite the doc comment to "addStrand runs `lyx reed add --cmd <command> --role <role>` in the hub and returns the strand's name and guid." with no second line about issue 673.
---

### [LOW] addStrand's #673 workaround is stale after the parent's anchor-default change

**Location:** `internal/creelcli/smoke_helpers_test.go:86`

**Issue:** The task's D9 added `--anchor own-window` to the smoke helper's `reed add` as the #673 workaround, and its comment says the default anchor "splits it into the selvage shell window and draws its pane too short".
The parent's merged change (`internal/reedcli/add.go`) made `own-window` the flag's default, removed `below-parent` from the CLI vocabulary and deleted the CLAUDE.md "Workarounds" section that required the flag.
Both sides merged without a conflict, but the helper's comment now describes behavior that no longer exists, and the explicit flag does nothing.
The decision record's open risks already say the flag is removed once the placement fix lands, so the cleanup is due now.
The argument is harmless to run (`own-window` is still valid), so this is not a break.

### Checked, no finding

- `AcceptStartupGate`, `NoInputBox`, `startup_test.go`, `seam_enforcement_test.go` and `session_test.go` assume no below-parent or shell-window anchor; the task's new rows use no anchor, and the parent's `AnchorOwnWindow` and `AnchorHidden` substitutions in the other shuttleengine tests touch different tests.
- `internal/shuttleengine/doc.go` and `pattern/PATTERN-daemon-wakeups.md`: the parent changed neither merged text region the task edited, and the parent changed no pattern file; no contradiction with the merged code.
- `cmd/lyx/testdata/test-budget.yaml`: the parent's `contracts/stencils`, `languageconfig` and `orchcli` bumps and the task's `internal/shuttleengine` 179 to 180 bump both survive; the top-level test count of shuttleengine moved by one on the task's side alone and the parent added none.
- `internal/creelengine`, `internal/creelcli` and `internal/testkit/creelfake`: the parent did not touch them, and the shuttleengine contracts they call (`IdleProbe`, `AcceptStartupGate`) were not altered by the parent.
- Test economy: the parent added or rewrote no shuttleengine test that duplicates `TestRunner_AcceptStartupGate` or the new `SessionIdle` rows, and both `NoInputBox` readings (readiness and pane fallback) have rows.
