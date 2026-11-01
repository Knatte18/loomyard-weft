---
round: 1
focus:
  - "Scope against the board entry's Idea: the entry lists reading the refused turns, closing the rest through steer wording, stencil instructions or a guard answering with the allowed command, and trialling the same rule as a `CLAUDE.local.md` heddle line. Settle which of these the diff makes, and whether any change in the diff (plan-spec wording, `docs/overview.md`, `doc.go`) is licensed by the entry or by an `Added after Discussion` entry."
  - "`internal/shellguard/rows.go` `compileCheckSentence` and `refusedRaw`: settle whether the steer ever offers a compile check that the hub's `refuse_raw` prefixes refuse, including prefixes of different word counts, a prefix longer than the form, a blank prefix, and a prefix that names flags or `./...`. Settle whether an empty or all-refused list leaves the steer well formed."
  - "Raw go row's steer and notice against the raw-go pattern: settle whether `go build ./internal/x` and `go vet ./internal/x` are in fact allowed by the row's deny patterns, so the offered command is not itself refused. Check `rows_integration_test.go` for the cases that pin this."
  - "Python row's `steerPythonDeny` and `noticePythonDeny`: settle whether the new text keeps the init-time forbidden-character checks, still carries the edit rule the docs say the steer carries, and whether the claim that a version check is refused matches the python row's pattern in command position."
  - "`internal/shuttleengine/claudeengine/testdata/settings-golden.json`: settle whether the golden's churn is exactly the regenerated steer and notice text and nothing else, with no unrelated key or ordering change."
  - "Doc and spec claims against the code: `docs/overview.md`'s shuttle paragraph, `doc.go` and the `loom-plan-spec.md` compile-criterion wording versus the behavior in `rows.go`. Settle any claim the diff does not bear out, and whether the commit message's account matches the diff."
  - "Test economy in `rows_test.go`: settle whether `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` duplicates an existing steer or notice test, and whether the python wording change has coverage beyond `TestCheckTexts`."
---

Seed focus only; it caps nothing and states no verdict.
The diff is small (one commit touching the shell guard rows, their tests, the golden, and three doc lines), so the base diff review stays the main work.
