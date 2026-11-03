```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1, F3, F4 and F5 and check the fix covers every member.

**Findings**

- LOW — `internal/shuttleengine/interruptguard.go:1`. The header says "every interrupt key shuttle sends into a pane passes the pane guard here". The new `dismissPicker` in `send.go:343` plays Escape through `playInputs`, outside this file. The F1 fix updated `PATTERN.md` and the pattern file but left this twin claim unchanged, and the pattern file now allows a second exception. A reader of this file would be misled.

**Sets checked, no gap**

- F1: the `PATTERN.md:66` line, the pattern file's lines 3, 26, 32, 38 and 44, and the new dismissal bullet cover the interrupt-guard claims. The rest of the repo-wide grep hits are unrelated "one exception" phrases. `internal/orchcli/session.go:80` describes `Interrupt` only, so it is not stale. `shuttleengine/doc.go` and `claudeengine/doc.go` hold no other "only via guards" claim.
- F3: every `noBoxReprompts` write sits at `wait.go:347`, `wait.go:556-558`, `wait.go:582` and `send.go:922`. Rows 1, 2 and 4 of `TestGate_NoInputBoxBoundCountsHoldsInARow` pin those sites, except `wait.go:582`. The fixer's mutation run showed this directly. Row 3 passes with `wait.go:347` removed because `wait.go:582` still resets, so it pins the two resets only together. That is not a gap.
- F4: the only non-test sites that derive `NoInputBox` are `send.go:209`, `conditionedsend.go:71` and `session.go:279`. `send.go:209` now reads the new `busyHold.noInputBox` field. The other two derive from `readiness`, which `dismissPicker` does not touch. The failing-key hold is the only `busyHold` built without going through `readiness`, so the set is covered.
- F5: `conditionedsend.go:71` now has an asserting row.
- F2, F6: not set-based; I did not check them.

**Calls**
1. A grep for `readinessReasonNoInputBox` and `noBoxReprompts` in production code and for exception or guard wording in `.md` and `.go` files: needed the full site sets, which my context did not hold.
2. A grep for guard and Escape wording in `doc.go`, `interruptguard.go` and `claudeengine/doc.go`: to find twin claims of the interrupt-guard rule.
