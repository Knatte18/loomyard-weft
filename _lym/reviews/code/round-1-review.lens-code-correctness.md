```yaml
lens: code-correctness
outcome: reported
```

**Lens list (code-correctness)**

- `Verdict`, no `scope.json` on disk: returns `"", nil` and creates no directory. Correct.
- `Verdict`, stat fails with something other than not-exist: returns a wrapped error. Correct.
- `Verdict`, file present: `ReadScope` returns `found=true`, so it renders. The `!found` branch only covers a file deleted between the stat and the read, and returns `"", nil`. Correct.
- `renderVerdict`, scope off: the heading and the scope line only, with no compare lines. Correct.
- `renderVerdict`, partial scope with an empty `Skip`: renders `Scope: partial, skipping .`, which is malformed. **Finding F-A, LOW**, see below.
- `renderVerdict`, waived record that still holds old compare fields: shows the Baseline line, then `Compare: waived: …`. Acceptable, since the waiver is checked first in `compareText`.
- `renderVerdict`, an `Acceptance.By` outside the three constants: renders `accepted by : reason`. The set is closed, so this is not reachable in practice.
- `recordCompare` from `pass`: runs after `Compare` succeeds and before the `Passed` branch, so a pass and a failure are both recorded. It stores `capture.Head` as the baseline and `Head()` as the compared commit.
  - A `Compare` error returns before the record is written, and the driver re-steps.
  - A `Head()` error returns an error, with no stale `ComparePassed` written.
  - A `recordCompare` failure after a successful compare returns an error, and the re-step compares again. Correct.
- `runAgent`, Done case: the hard-coded `true` is right, because the agent's `compareEntry` gate is the only route to Done and it passes only on `result.Passed`.
  - `told.BaselineHead` is `capture.Head` on both the fresh and the interrupted-agent paths.
  - The `scope` read before `recordCompare` only feeds the `Unruled()` check, which the record does not touch. Correct.
- `runAgent`, Stuck case: it keeps the compare result `pass` recorded, which is `passed=false`. After accept or withdraw and a resume, `pass` compares again and overwrites it. No stale passed record. Correct.
- Scope replacement (`baseline.go:145`, `verbs.go:85`): both construct a fresh `ScopeRecord`, which zeroes the compare fields, so the section reads "Compare: not run." Correct, though I only saw the constructor lines.
- `withSection("\nChange body\n", v)`: gives `"\nChange body\n\n" + v`.
  - An empty section returns the body unchanged.
  - A whitespace-only body returns the section alone. Correct.
- `Finalize`: it mutates only the local `summary.Body`, and `Call` re-parses the file each entry, so a re-run cannot double the section.
  - `LandingMessage` trims the leading whitespace and appends the credit block after a blank line.
  - `bodyWithCredit` trims the body. An empty body with a section gives the title plus the section. Correct.
  - A malformed `scope.json` makes the verdict read fail, which is a hard error before any merge. That matches the stated design.
- `PRReview.openPullRequest`: `bodyWithCredit(withSection(...), credit)` gives body, section, credit, in that order, and a verdict error stops before any GitHub call. Correct.
- `sessionSpec` and `DescribeTable`: an empty verdict becomes `noDriftVerdict`, so `stencil.FillWith` never gets an empty marker. Correct.
- `driftVerdict()` with a nil seam: returns `"", nil`. A seam error is wrapped. Correct.
- `DescribeTable` wiring, `wiring.go:816`: runs once per table build, inside the closure, so it is read at call time. A verdict error aborts the table build. Correct.
- `landingDeps`: the closure runs `Verdict` over `l.AnchorPath()` at call time. Correct.

**Findings**

- **F-A, LOW, `design`:** `internal/driftengine/verdict.go:176-184`. For `ScopePartial` with an empty `Skip`, `scopeText` renders `partial, skipping .`. Whether that record can exist depends on config validation, which I did not check.
  - Fix: render `partial.` when `len(scope.Skip) == 0`.
- **Unconfirmed, not a finding:** the section starts with `## Regression check` and is passed in the landing commit message. If the merge commit is made with a `git commit` cleanup mode that strips `#` lines, the heading would vanish from the commit message. My one grep of `fabricengine` and `gitrepo` did not show the cleanup mode. The handler may want to check how `MergeOptions.Message` is committed.

**Tool calls**
1. `grep` in `internal/fabricengine` and `internal/gitrepo` for message cleanup handling. I needed it for the `#`-heading question, and the context did not hold the merge-commit code. It was inconclusive.
