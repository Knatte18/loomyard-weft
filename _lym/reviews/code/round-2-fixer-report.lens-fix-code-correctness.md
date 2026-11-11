```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I traced `renderVerifyGateFindings` with a report of Attempt 2 and Cap 3. The header now reads `Verify gate failed (evaluation 2; the session may be re-prompted 3 time(s)).` The same fields give "evaluation 4; ... 3 time(s)" when Attempt runs one past Cap. The two numbers can no longer contradict each other.

- **Code path:** the format call has two `%d` verbs for the two arguments, `r.Attempt` and `r.Cap`. The change touches no other branch, and every early-return branch writes the header first.
- **Tests:** each updated assertion matches the new text:
  - `"evaluation 2; the session may be re-prompted 3 time(s)"` is a substring of the output for Attempt 2, Cap 3.
  - `"evaluation 1; ..."` matches the Attempt 1 cases.
  - The `HasPrefix` checks on `"\nVerify gate failed (evaluation "` match the header placed after the way-forward line.
  - Each of these fails against the old wording, so the tests do check the change.
- **Callers:** `renderVerifyGateFindings` has the same signature. In `wrapRunLevelEntry`, `Cap` comes from `entry.Attempts`, which matches the "re-prompt budget" comment on the field.
- **Stale text:** my search for the old wording found no other consumer. The only hit is `docs/research/.../calls.tsv`, a captured historical measurement.

I did not run the F2 stencil change, since it is prose only and the Go code does not parse it.

Further tool calls (1):
- `grep -rIn` for the old "attempt N of M" wording across the repo, because the fixer's earlier grep covered only `.go` and `.md` files.
