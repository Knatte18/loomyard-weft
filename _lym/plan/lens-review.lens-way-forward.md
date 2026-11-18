```yaml
lens: way-forward
outcome: reported
```

The lens: way-forward. For each refusal, stop or recovery step the cards add, I checked whether the next step it names can succeed from the state it leaves.

**Findings**

1. **MEDIUM — Card 9, the reach turn's error path.** A `Pending` or record error becomes the gate's own error. `ReachPendingOf` can fail on an unreadable `reach.json` or on a `ReachReportMatches` call that cannot open fabric or read the merge base. Either way, that error ends the strand's gate wait as an infrastructure fault. The way forward is to re-step the row, but a corrupt `reach.json` fails every arrival again, and nothing tells anyone to delete the file or re-run the gate. That turns an advisory note into a stop that cannot progress, against D3's "It is advisory".
   - Evidence: `internal/shuttleengine/wait.go:567-569` (`evaluateGate` error → `shuttle: gate: %w`) and `:531` ("evaluateGate's returns are infrastructure faults").
   - Fix: `ReachPendingOf` answers false for a report it cannot read or match, and logs a Warn. Only a failure to write the listed file stays a gate error.

2. **MEDIUM — Card 10, the `--since` refusal.** `FirstParentCommits` matches `since` against the walked commit SHAs. A short SHA, a tag or `HEAD~3` is never met, so the walk ends with "not on the branch's first-parent history; pass one that is". A caller who passes the same rev again gets the same refusal, so the way forward cannot succeed.
   - Evidence: card 10 Intent, the `FirstParentCommits` bullet and the "`--since` the walk never meets" refusal.
   - Fix: resolve `--since` to a full commit SHA through fabric before the walk. A rev that resolves to no commit gets its own refusal: "pass a commit SHA or ref that names a commit".

3. **BLOCKING — Card 10, the branch walk itself.** `Fabric.FirstParentCommits(branch, …)` is said to read "through gitrepo's log with FirstParent". `Repo.Log` refuses any tip that is not a SHA, so every walk over the parent branch, `--branch` or the current branch errors before it lists anything. The card names no step that resolves the branch to its tip SHA, and no refusal for a `--branch` that names no branch.
   - Evidence: `internal/gitrepo/historyreads.go:150-152` (`if !validSHA(tip) { return nil, ErrInvalidSHA }`).
   - Fix: resolve the branch tip SHA first. Add a refusal for an unknown `--branch` that names the way forward, and give it a `TestGateReach` row.

4. **MEDIUM — Card 10, `--test` with an unmapped or malformed test.** Two inputs have no stated outcome or refusal:
   - a `--test` value that is not `<package>.<Test>` (no dot);
   - a test no map holds (`FuncsReachedBy` returns false).
   
   Without a stated outcome, a walk of up to 200 commits can report "0 commits listed" for a test that was never mapped, which wrongly says that no landing is to blame. No `TestGateReach` row reaches either input.
   - Evidence: card 10 Intent, the `--test` form and its refusal list.

5. **MEDIUM — Cards 10 and 14, a scoper other than `go`.** Scope "Out" requires the new verbs and steps to "write a note naming the scoper for any scoper but `go`". Card 6 covers this for the step. Neither `lym gate reach` nor `lym gate mutate` names that branch or its way forward, and no test row reaches it.
   - Evidence: decision record, Scope Out bullet 5; card 6 does cover the step.

6. **LOW — Card 2, an unreadable map.** `Open` passes over a map holding only a `BuildFailure` or a bad index. A map file that does not parse is not covered and presumably fails `Open`. One corrupt shared map would then disable every reach note, triage line and verb hub-wide, and the error names no way forward such as "rebuild it with `lym gate test --map` from the prime".
   - Evidence: card 2 Intent, the `Open` bullets.
   - Fix: pass such a map over, or put the package and that way forward in the error.

7. **LOW — Cards 6 and 14, unrefreshed and not-judged lines.** Only the uncommitted-files reason names a way forward.
   - A package past `reach_budget_min` could say "run the gate again": already-refreshed packages cost nothing on the next call, so that step progresses.
   - In the prime, a shared map that is stale and fails `Fresh` is "not judged" in `lym gate mutate` with no next step, though "rebuild the maps with `lym gate test --map`" would progress.
   - Evidence: card 6 Unrefreshed bullet; card 14 Fresh-maps bullet.

8. **LOW — Card 4, the guard test's failure text.** `TestCoverageMapReaderBoundary` fails "naming file and line" but states no fix: read through `internal/testreach`, or pin a new single-package reader with its reason.
   - Evidence: card 4 Intent.

Not in my lens: the reach turn's listed file is keyed by site label. Code-Burler-Fix's writer can therefore get a turn for a function Darn-Write's writer was already shown. That may be intended, since they are different writers.

Tool calls:
- One grep of `internal/shuttleengine/wait.go`, `internal/fabricengine/warpforward.go` and `internal/gitrepo/historyreads.go`. My context lacked what the shed driver does with a gate error, and whether `Repo.Log` takes a branch name as its tip.
