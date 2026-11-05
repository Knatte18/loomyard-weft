```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness. For each changed function or branch I ran one concrete input and recorded whether it was correct.

**Per-function list (case run, verdict)**
- `ResolvePullRequest` with a beam: `BeamWithPullRequest` and a parent in `on` mode gives a yes, so correct. `BeamWithoutPullRequest` with `on`/`off` gives a no, so correct. Parent `merge` gives a yes with `MergeAtOnce`, so correct. A beam with no task commits gives a no before any origin check, so correct.
- `Finalize.recordedBeamLanding`:
  - Open recorded pull request: squash, no title or body edit, record marked for dropping. Correct.
  - Closed recorded pull request: halts naming the reopen and `lym loom resume`. Correct.
  - Merged at the task head: already-landed path. Correct.
  - Merged at another head: see F2.
- `Finalize` drop (`landed`, `dropBeamRecord`): the drop happens only after `pushParent` returns `Done` with no error. A push halt keeps the record, and a resume re-enters through the merged path. Correct. A failed drop is only logged, as D8 allows.
- `mergeRefusalReason` for a beam: `RefusalOther` and `RefusalMethod` name GitHub's message, `lym beam pr status` and `lym loom resume`, never `batten cancel`. Correct. `RefusalPending` is retried in `mergeOnGitHub` within the one-minute window. Correct.
- `openAtOnce` for a beam: a found pull request gets no `Edit`, and a created one gets the description's text. Correct.
- `Finalize.pullRequestLanding`, scratch-record override: see F1.
- `ReadMergeState` / `mergeCauses`: `unknown` is retried with a doubling delay capped at the remaining window, then reported as a cause. A merged or closed pull request returns after the state cause plus a head-mismatch cause. An empty `mergeable_state` gives a "not one that merges" cause. Correct.
- `judgeReadyLanding`:
  - Refusals 1 to 6 exist and each names its way forward.
  - Merged with a head differing from the beam's and `MergedLacks` empty passes. Correct.
  - Closed, head mismatch and not-mergeable each name the verbs D3 asks for. Correct.
  - The no-pull-request branch: see F3.
- `ParentUnmerged`: the upstream is fetched only when it exists and the push skip is unset. It checks both the local and upstream tips with `IsAncestor`, and skips an empty tip. Correct.
- `MergedLacks`: `BeamLineCommits` returns the line oldest first (`slices.Reverse`), so `line[i+1:]` is the commits past the merged head. Records-only commits and parent merges are skipped, and a merged head not on the line lists the whole line (refuses). Correct.
- `prOpen`, `openRecorded`, `prUpdate`, `prClose`, `prStatus`:
  - `prOpen`: a recorded open pull request answers `already`, a closed one is reopened, and a merged one is left for Finalize.
  - `prOpen`: a failed record write after `Create` names the pull request and the rerun. A found pull request gets no edit.
  - `prClose`: a failed `Close` does not drop the record, and an already closed or merged pull request skips GitHub. The comment depends on the resolver's answer.
  - `prUpdate` and `prStatus`: all correct.
- `BeamAwait.settleEntry` / `recheckStanding` / `landOrRefuse`:
  - Fresh first entry with no ready record falls through to `enter`.
  - Status Finalize, no request, ready standing: re-check and `Done`.
  - Status Finalize, ready not standing: the hold is lifted and the dobby started.
  - A failed status write or ready read is retried at the next wake.
  - A failing re-check lifts the hold, sets `entered` and writes the await marker.
  - All correct.
- Wake stamp and driver (`facts`, `lastRestepAt`): the stamp is written after each wake that did not end the row. The repair count resets only when the stamp is at least 10 minutes past the last re-step, and only when the count is above zero. Bounded by the cap, and recipe-agnostic through `CompletedWake`. Correct.
- `shedengine` retired move: a retired row (not a row of `Producers`) calls `RetiredMove`, then persists paused on the target, and any other unknown row still refuses. On a move error nothing is persisted and the next step repeats the move. Correct.
- `MoveOffRetired`: a crash after the carry and scratch removal, then a retry, finds the landing record already carried and commits it. The move is idempotent apart from the notice (F4).
- `beamForkPrecondition`: ready true, a ready read error and ready false each give the right refusal or pass. Correct.
- `holdReadyBeam` / `noticeReadyBeamMembers`: a read error holds starts and posts nothing. A failed post is retried, and a state-write failure only warns. A not-ready reading clears the episode. Correct.
- `announceLandings`: no pull request recorded does nothing. A failed post returns before recording the head, so it is retried. A records-only advance records the head with no line. A missing announce file falls back to the record's head. Correct.
- `ownLandingHold` / `beamUp`: Finalize and Beam-Down, and a retired row with the own run not done, are the beam's own landing hold. The orch comes up and the dobby does not. Any other hold is still refused. Correct.

**Findings**

- **F1 — MEDIUM, design** — `internal/landingshed/finalize_pr.go`, `pullRequestLanding`, the `beam == BeamWithoutPullRequest && found` override, with `landingForDecision`'s `StateClosed` branch.
  - Any scratch pull-request record found for a beam is merged "as it stands", with no check that the pull request is open, and the resolver is skipped.
  - D8 limits this to a scratch record holding an **open** pull request.
  - With `MergeAtOnce` true, a closed scratch pull request (for instance, one the operator closed on GitHub to land without) takes the `reopen: true` branch. Finalize reopens it and merges it, even under an `on`/`off` parent where the resolver would have closed it and landed locally.
  - A merged scratch pull request falls through to the already-landed path, so the resolver's other inputs are never read for it.

- **F2 — MEDIUM, design** — `finalize_pr.go`, `recordedBeamLanding`, merged-at-another-head branch (`pr.CommitsMissing(status.HeadSHA, head)`) against `internal/beamcli/deps.go` `MergedLacks`.
  - `lym beam ready` passes a merged pull request when the beam is past it only by parent merges and records-only commits. `MergedLacks` skips both.
  - Finalize's test is a plain `CommitsMissing(mergedHead, taskHead)`, and I did not see it exclude parent merges. A beam that ready passed for this reason is then treated as "lacking commits" and decided as with none recorded.
  - Under a required or `merge` parent, that path finds or opens a new pull request and merges it, which D3 and D8 intend to avoid ("Finalize lands it").
  - Unverified: whether `CommitsMissing` or the point where `pullRequestLanding` runs (before or after the parent merge-in) already accounts for this.

- **F3 — LOW, design** — `internal/beamcli/land.go`, `judgeNoPullRequest`. The `decision.Halt != ""` branch words the demand as "GitHub's rules … require a pull request" unless the parent mode is `required` or `merge`. A halt also comes from an origin on github.com that `ParseOwnerRepo` refuses, which `ResolvePullRequest` always halts on. With an `on`/`off` parent and no rule, the refusal then names a demand that does not exist.

- **F4 — LOW, design** — `internal/pawlengine/retiredmove.go`, `MoveOffRetired` / `retiredMoveNotice`. A move retried after `RemoveScratch` succeeded finds no scratch record, so `carried` is nil and the notice omits "its pull request stays open". This happens when a later step (a file removal, the commit, or the status write) failed and the move is repeated.

**Tool calls**
1. `sed`/`grep` of `internal/fabricengine/beamline.go` and `mergeOnGitHub` in `finalize_pr.go` — the manifest lacked `BeamLineCommits` ordering and the merge loop.
2. `grep` for `pushParent` and `dropBeamRecord` call sites, plus `AnchoredPathHasContent` — to confirm the drop covers every success exit and that a deleted tracked file still commits.
