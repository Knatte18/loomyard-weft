```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches behind F1, F3, F4 and F6, and check the fix covers every member.

**Findings**

- LOW: `contracts/stencils/parentdirective_test.go:5` is a stale twin of F1, and the fix left it as it was.
  - The grep shows the comment still reads "the discussion role's interactive questions come from the…".
  - That is the same one-interactive-role claim `pattern/PATTERN-parent-directive.md:5` had before the fix, and it now misdescribes the code. The Drift-Verify seat under `verify_interactive` fills `{{.mode_rules}}` with operator text too.
  - I did not read the comment in full, only the matched line. Its wording should be updated to match the PATTERN entry.

**Checked, no gap**

- F1 (docs claiming one interactive role): the only other hit in `pattern/`, `contracts/`, `internal/` and the `.md` and `.go` files is the test comment above.
- F3 (sites that feed the recorded mode to `ResetRuns`): `ResetRuns` has one non-test caller, `verify.go:509`, which now takes the spawned table's `Interactive`. No other `d.Config()` read feeds the recorded mode.
- F3 and F5 (test fakes): the one fake `VerifyTable` in `verify_test.go:167` now sets `Interactive` from the config, so the operator-session case at `verify_test.go:808` still gets operator true.
- F4 (other "non-interactive" claims about the drift agents): grep finds none left. `doc.go:62` and `doc.go:103` already state the interactive case.
- F6 (the autonomous-text claim):
  - `seats_test.go:247` asserts the autonomous text contains "giving the reason the drift is intended", which still holds after the first line was restored.
  - The stencil opening line "one session that judges" is shared by both modes, so it cannot say "autonomous" again. The claim in `_lym/landing/summary.md` is outside the fixer's write surface and needs the parent's correction.
- `template.yaml:6`, `cli.go:120` and `cli.go:154`, which cover `verify_interactive` or the operator session, already describe the interactive mode.

**Calls made**
- Grep of `pattern/`, `contracts/`, `internal/` and `PATTERN.md` for "non-interactive", "one place a question" and "discussion role" claims, to rerun the F1 and F4 sets.
- Grep of `VerifyInteractive` and `verify_interactive` repo-wide, to rerun the F3 and F4 sites and find other docs.
- Grep of `ResetRuns(`, `OperatorSessionOpen(`, `autonomousVerify` and "you judge intended", to rerun the F3, F5 and F6 callers and text.
