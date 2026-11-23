```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for a test row that would fail if it broke.

**Behaviours and the tests that pin them:**

1. `contracts/orchrole/role-intake.md:30-38` gives the intake fork a fourth outcome, a close as already fixed: judged from the main branch's code and history, closed with `"completed":true`, and the reason names the fixing commit. If the fork cannot tell, the issue goes on the board. Pinning test: none. This is static prose in the orch role's on-demand file, so it is exempt.
2. `internal/boardcli/intake.go:117` changes the `close` command's `Short` to "noise or already fixed". Pinning test: none. This is static help prose, so it is exempt.
3. `internal/boardcli/intake.go:128-132` adds a `Long` help sentence and a second example, `{"issue":15,…,"completed":true}`. "Example" became "Examples". Pinning test: none on the wording, which is exempt. The keyword checks in `internal/boardcli/help_test.go` match "Example", which is a substring of "Examples", so the rename cannot break them. The new example uses only the keys `issue`, `reason` and `completed`, which the existing `Use` token declares. `TestPayloadUse_MatchesDeclaredKeys` (`internal/boardcli/payload_test.go:41`, row "intake close") checks `Use` against `intakeClosePayload`, and `Use` did not change.
4. `plugins/ly/skills/board/SKILL.md:231,247` adds the already-fixed close example and a bullet. Pinning test: none. This is static skill prose, so it is exempt.
5. Closing with `"completed":true` is not new runtime behaviour: the `completed` field and `intakeClose` existed at the review base, and the diff does not touch them. Nothing new needs a pinning row.

The mutation report agrees: it ran no mutants and found no uncovered changed blocks, because the diff changes no executable line.

**Findings:** none. The diff adds no Go behaviour, and every changed line is static prose, which this lens exempts.

**Tool calls:**
- I read the opening of `internal/boardcli/payload_test.go` and grepped `internal/boardcli/*_test.go` for "Example". My context named `TestPayloadUse_MatchesDeclaredKeys` and the "intake close" row but held neither the test's body nor the help tests' "Example" assertions.
