```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each carried-out proposal (F1–F5) and found two weak rows in the proposals as applied. One more call: I read `applyRenumber` in `internal/planparser/renumber.go:316-385` to learn the step order.

Findings:

1. **LOW** — `internal/planglyph/index_test.go`, row "format over a test file whose base name only another directory holds" (F4's third proposed row).
   - **Summary:** The row does not pin the `path.Base` match it was meant to pin.
   - **Evidence:**
     - The card names `internal/x/x_test.go#`, but the fixture holds only `internal/y/x_test.go`.
     - `glyphlessFile` calls `TOC(worktreeRoot, "internal/x/x_test.go")`, which targets a path the repository lacks. That is the same input as the "repository lacks" row, so it returns the toc-error path or an answer with no entry.
     - The `path.Base` loop never meets the other-directory entry. Changing `entry.Name == path.Base(file)` to a looser or wrong comparison would leave the row passing.
     - A row that reaches the clause needs a toc answer listing a same-base entry that is not the requested file. For a file target, TOC probably does not return one.
   - **Fix direction:** drop the row, or reword its name so it does not claim to pin the `path.Base` match.

2. **LOW** — `internal/planparser/renumber_test.go`, row "insert: write of a card changed only by a bare reference" (F2's proposed fault).
   - **Summary:** The row passes even if the bare-only card is not journaled.
   - **Evidence:**
     - In `internal/planparser/renumber.go:359-361` the stayed cards are written after the moved cards and before the overview.
     - Faulting `WriteFile` on `01-alpha.md` fails before alpha's bytes change, so `01-alpha.md` is identical afterwards whether or not it was journaled.
     - The tree is restored either way, because the moved cards are restored from the journal. The `len(refs) > 0` journaling clause (`renumber.go:299-302`) is therefore not pinned.
     - Only the second added row, which faults the overview write after alpha has been written, fails when the journaling is removed.
   - **Fix direction:** keep the overview-fault row as the pin; the alpha-write row is redundant with the existing "moved card write" rows.

No other findings.
- F1 and F3 are right: the new `Spans` and `Pair` lines and the delta `Summary` reference are each reached by a row, and the explicit `Intent` stops the default-to-`Summary` rewrite from firing.
- The gate was green.
- F5's two-sentence rewrite is accurate.

Tool calls:
1. `grep` on `applyRenumber` in `internal/planparser/renumber.go` — context lacked the step order.
2. `Read` of `internal/planparser/renumber.go:316-385` — context lacked the step order past line 800 of the earlier span.
