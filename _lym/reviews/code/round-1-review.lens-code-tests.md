```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it, or none** (all rows are in `TestFirstMutatingGitCommand`, `internal/burlerengine/cluster_test.go`)

| Behaviour | Pinning row | Verdict |
|---|---|---|
| Unquoted `>` plus its target dropped (`2>/dev/null`) | line 380, and the chain at 385 | Pinned. Without the drop, `/dev/null` reads as a branch operand and the row flips to mutating. |
| Spaced redirect `> file` drops the target | line 382 | Pinned. |
| Attached fd digits dropped before the operator | lines 380, 381 | Pinned. A leftover `2` would read as an operand. |
| Redirect before the subcommand hides no mutating verb | line 383 | Pinned. |
| Quoted `">"` stays an operand | line 384 | Pinned. |
| Issue #898 chain is not mutating | line 385 | Pinned. |
| `&` attached after the operator consumed (`>&`, `2>&1`) | none | **Unpinned.** Row 381 passes even without the `&` consumption: the `&` ends the segment, `1` lands in a non-git segment, and `git branch` is still judged read-only. |
| `dropNextWord` reset in `endSegment` (redirect as the last token before `;`, `&&` or newline) | none | **Unpinned.** No row has a redirect with no target before a separator. |
| Digit word not attached to the operator stays an operand (`git branch 2 > x`) | none | Unpinned. |
| Backslash-escaped `\>` stays in its word (D1) | none | Unpinned. The grep finds no escaped-redirect row. |
| `<` input redirect, repeated operators (`>>`, `<<`), `>|` | none | Not a gap. `<` shares the `>` case, and the repeat loop changes no verdict that I could construct. |

Row economy: rows 380 to 385 each cover a distinct behaviour, so I found no duplicates.

**Findings**

1. **MEDIUM, design.** Location `internal/burlerengine/cluster_test.go:381`, code at `cluster.go:422-424`. The `&` consumption after a redirection operator has no failing row.
   - Without it, `git 2>&1 commit -m x` splits at the `&` into `git` and `1 commit -m x`, so the mutating command is missed.
   - A row `git 2>&1 commit -m x` expected true would pin it, and would also cover D2's "redirection that cannot hide a later mutating subcommand" for the `&` form.
   - Evidence: `cluster.go:422-424`, `cluster_test.go:381,383`.

2. **LOW, design.** Location `internal/burlerengine/cluster_test.go:385`, code at `cluster.go:380` (the `dropNextWord = false` in `endSegment`). Nothing pins the reset when a redirection has no target before a separator.
   - Without the reset, the first word of the next segment would be dropped.
   - A row such as `git branch -a >; git commit -m x` expected true would pin it.

3. **LOW, design.** Location `internal/burlerengine/cluster_test.go:380-384`, code at `cluster.go:412-415`. No row covers the fd check against a digit word that is a real operand, which is D1's "only the redirection target leaves the words" bound.
   - A row `git branch 2 > x` expected true would pin that a detached digit stays an operand.

4. **LOW, design.** Location `internal/burlerengine/cluster_test.go:384`. D1's backslash-escaped `<` or `>` staying in its word has no row.
   - Only the quoted form is pinned.
   - A row `git branch \>` expected true would pin it.

**Tool calls**
1. `grep` over `cluster_test.go` for escaped or compound redirect rows, to check whether `\>`, `>&`, `<<`, `>>` or `>|` rows already exist. My context only held the diff's rows, not the whole table.
