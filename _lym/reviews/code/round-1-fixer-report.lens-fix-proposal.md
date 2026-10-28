```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F3, F4 and F5 as the fixer carried them out, under the fix-proposal lens.

No findings.

- **F1:** I built the case where the proposed test passes with the defect in place, and it does not hold.
  - Without `isBoardOverview` treating `project-x.md` as an overview, `isBoardPath` rejects the path. The commit then falls back to the git-level replay, and `board.json` conflicts because both sides retitled it. The integration row would fail in that case.
  - The `TestIsBoardOverview` rows match the code's behaviour. `project-.md` leaves an empty slug, `project-a/b.md` and `dir/project-x.md` are rejected by the slash check, and `project-x.json` fails the `.md` suffix cut.
  - The test is a tier0 internal-package file, matching the existing `mergepaths_test.go` pattern.
- **F3:** The `ghost` dependency is a slug with no board entry, so it takes the `codeSpan` branch of `afterCell`. It does not change layering. The pinned cell ``T2<br>T7<br>`ghost` `` follows `depends_on` order, with the done `shipped` dependency omitted. The tier0 and tier1 gate runs passed.
- **F4:** `project-tidy.md` uses the draft task `chore` and renders `Ready (0)` with `_None._`, then a `Draft (1)` table. That is the expected "after Ready, not under Ready" pin, and the gate passed, so the golden text matches what is rendered.
- **F5:** Both comments point to the other constant without naming its symbol, and neither package can import the other. This stays correct if either constant is renamed.

Tool calls made: none.
