```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness. I ran the cases below through the diff's changed functions.

**Per function, case run, result**
- `reopen` (`land.go`): task slug, done beam, owner elsewhere, no pawl run, Beam-Await, Finalize, done run, PR-Review, Parent-Align, Beam-Describe and a halted run all give the right refusal or request. A failed `Unready` after `WriteReopen` leaves the request written, and the message tells the operator to rerun. Correct, with the caveat in F3.
- `RemoveReady`: absent record, record naming another slug, and a normal record are correct. The read-then-lock window is benign.
- `answerReopen` (`prreview.go`) on a nil seam, no request, no record, an open PR, a draft PR, `Close`, a closed PR and a transient `Status` failure is correct. A failure between `SetDraft`, `Comment` and `Close` re-steps without a duplicate GitHub action: the next step reads the new state and skips what is done. It is wrong when the PR is merged (F1).
- `openPullRequest` draft marking: a re-entry on a draft calls `SetDraft(false)` before the edit. The stale `entryStatus` passed on to `wait` only affects `Draft`, which nothing downstream reads. Correct.
- `wake`: `answerReopen` runs after `holdWhileCovered`. Correct.
- Beam-Await `answerReopen`: request for this run with a dead dobby, a live dobby, a failed start, a `DobbyLive` error, a request for another run, and a failed removal. Correct for all but one case (F2).
- `readReopenRequest`: `state.ReadJSON` tolerates a missing lock directory (`internal/state/state.go:99-105`), so task pairs with no `.lym/beam` are safe. A seed that is absent or has a different `StartedAt` reads as no request. Correct.
- `beamCommits`: no commits added, parent merged in, member merged `--no-ff`, and head equal to the parent tip are correct. The first-parent walk breaks at the first commit outside `added`, which is sound because every ancestor of a base-reachable commit is also base-reachable.
- `beamCommitEntry` and `commitSummary`: the regexp on a title containing `(#12)`, an empty message, a CRLF body, a `#` heading paragraph, a body longer than 200 runes and a short SHA are correct. A body whose heading and text share one paragraph is skipped whole, a cosmetic issue only.
- `pullRequestCommitTitle` and `mergeOnGitHub`: correct.
- `SetDraft` and `graphQL`: a GraphQL `errors` array, a failing PR read and an HTTP error are all reported. Correct.
- `Unready`: see F3.

**Findings**

- **F1, MEDIUM, design.** `internal/landingshed/prreview.go:677-709` (`answerReopen`).
  - A request seen while GitHub reports the recorded PR `merged` is neither refused nor closed. Without `--close`, the record is kept and reset, then the run reroutes to Beam-Await.
  - The next landing finds the recorded merged PR in PR-Review and takes the merged verdicts, so Finalize treats the new work as already landed, with no PR carrying it. The merged state is reachable when the PR is merged on GitHub inside the one-minute poll window, or when a resume follows a crash.
  - Fix: on a merged PR, Stuck with Halt naming the way forward, or remove the record as for closed.
  - Evidence: the `switch` at lines 678-694 has no merged case, and line 696 removes the record only for `Close` or closed.

- **F2, LOW, design.** `internal/pawlengine/beamawait.go:215-218`.
  - If `RemoveReopenRequest` fails, the code only logs and still posts the notice and starts the dobby. The request, which names the current run, stays on disk.
  - The next PR-Review reads it through `readReopenRequest` (`internal/loomcli/landingdeps.go:841-854`). It then reroutes again at once, drafting the PR and spending a reroute-cap slot, until the cap halts the run.
  - Fix: on a failed removal, keep the beam waiting and retry the removal at the next wake, as the land-request path does. At minimum, make the notice name the stuck file.
  - Evidence: `answerReopen` continues after the warn.

- **F3, LOW, design.** `internal/beamcli/deps.go:151`.
  - `Unready` always commits `LandingDirRel`, but `AnchoredPathHasContent` (`internal/fabricengine/commitweftpaths.go:104-106`) says git refuses a commit pathspec that matches nothing.
  - When a reopen at Parent-Align or Beam-Describe is repeated after the ready-record deletion was committed, and `_lym/landing` holds no other tracked or untracked file, the commit pathspec may match nothing. The verb then fails with "ready record could not be removed", although nothing is wrong. I did not confirm that `CommitRecordsPaths` omits this guard.
  - Fix: guard the commit with `AnchoredPathHasContent` and skip it when false.

- **F4, LOW, design.** `contracts/recipes/pawl-recipe.yaml:80` with `internal/shedengine/reroute.go:10-29`.
  - The reroute count resets only at PR-Review's latest Done or cap halt. PR-Review reaches Done only at the end of the landing, so a beam gets three reopens in total, and the fourth halts the run.
  - The summary admits this under "Known limits", and `lym loom resume` grants three more. It still contradicts "reopen so the beam can take new members" for a long-lived beam.
  - Fix: have the reopen answer reset the count, for example by recording a Done-equivalent history entry. Otherwise make the cap's way forward name `lym beam reopen`'s expected resume explicitly.

**Tool calls (3)**
1. Read `state.ReadJSON` in `internal/state/state.go`. My context did not show how it behaves when the lock directory is missing.
2. Read `CommitAnchoredPaths` and `AnchoredPathHasContent` in `internal/fabricengine/commitweftpaths.go`. My context did not show the pathspec-matches-nothing handling.
3. Grepped `internal/githubclient` and `pullrequest` for `BaseURL` to check where the GraphQL endpoint resolves. I raised no finding.
