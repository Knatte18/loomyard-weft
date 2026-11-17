```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked whether every refusal, step, row and test this fix introduced can be reached from the state it names and leads to progress.

**Findings**

1. **MEDIUM.** D3, the rewritten `lym burler validate-review` paragraph: "the verb resolves the worktree through `lymcwd` alone … from the seam cwd and never from the file's path", plus the fallback "or a seam cwd whose worktree does not resolve, gets the shape checks alone".
   - **Summary:** nothing says what the verb does when the seam cwd resolves to one worktree but the review file sits in another worktree's run directory. In that case the builder rebuilds the wrong context and the round checks give refusals that point nowhere.
   - **Evidence:** the file's path still decides whether it "sits in a run directory", but the worktree now comes only from the cwd. The only fallbacks named are "outside a run directory" and "does not resolve". A cwd in another worktree passes `lymcwd.ResolveWithin` (`internal/lymcwd/lymcwd.go:143`). The builder then takes that worktree's decision record path and anchor path. Its `decisions_routed` and digest checks run against a record that does not belong to the round.
   - **Fix:** the verb runs the round checks only when the run directory lies under the resolved worktree's own `_lym` run root. Every other file gets the shape checks alone. Add the mismatch case to the D3 acceptance row.

**Checked and found sound**

- The D2 held mark on a fix row that only notifies the parent can be answered. The notify line already names `lym loom decision add` as the way forward (`internal/shedadapters/burlerfix.go:105`).
- The new D2 row (a round N-1 record without `decisions_routed` gives today's text) can be reached.
- The F2 class refusal can be reached.
- In the D6 yield, the check only defers to code that already settles the round.
- The new D6 row can be reached through `softCapRule`'s accept at `internal/shedadapters/bouncer.go:1407`.

**Tool calls**

- Read `internal/shedadapters/doc.go` lines 600–620 and grepped `internal/shedadapters` for `decision add`. My context did not show whether the notify-only parent line names a way to answer a held finding.
