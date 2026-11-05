```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward the fix introduced in `decision-record.md`, as the fix-way-forward lens.

**1. MEDIUM: the D7 failed-capture Stuck names only a way forward that can loop.**
- Changed text: D7, "A current capture that fails or times out is a Stuck at Drift-Verify naming the capture's log and `lym loom resume` as the way forward".
- Summary: if the writer's code broke the capture (a build or harness failure), every resume reruns the same failing capture. This Stuck names neither the code-fixing route nor `lym drift defer`, and D12 does not admit `defer` here.
- Evidence: D7 names the code-fixing route (`lym loom goto --to Darn-Write`) only for unexpected cells. D12's admission list was widened in this fix for missing or changed baselines, a moved warp and a never-counted capture, but not for a failing current capture.

**2. MEDIUM: a baseline-digest or missing-baseline failure goes to fixers that cannot clear it.**
- Changed text: D8, "It fails, and `lym drift check` reports, when the counted baseline directory is missing or its digest mismatches (D7), naming the parent's `lym drift defer` (D12)". Also the D7 digest paragraph.
- Summary: at Darn-Write the entry's findings go back to the writer. At Merriam's gate they go to the fixer fork or `lym webster fix-verify` (D8). Neither can run `defer`, so each spends its attempts before the row stops and the parent can act.
- Evidence:
  - D8: "a failed `drift` entry at Merriam's gate is cleared through those same paths, so its findings reach the fixer's prompt".
  - D6 handles the same situation for a seal failure: "No session at the writer's row can run either verb, so the drift gate entry reports a seal failure without failing … and leaves the stop to Drift-Verify".
  - The fix gives the baseline failure no such route. It should fail without a fixer, or report without failing as a seal failure does.

**3. MEDIUM: `unsealed` and the recorded `diff`/`key_patterns` are written by a check that D8 says writes no tracked record.**
- Changed text: D6, "the first drift check that finds a run past Drift-Expect with no seal commit record `unsealed` in `scope.json`". D3, "records config's values at its next drift row or check and keeps them".
- Summary: "drift check" here covers the gate entry and `lym drift check`. D8 forbids both from writing `scope.json`, so the step cannot be carried out as written. If only rows record it, D15's forge window stays open for the writer's whole session.
- Evidence:
  - D8: "it runs no judge and writes no tracked record".
  - D8 Bound: "`check` and the entry write only their scratch capture directory".
  - D8 acceptance criterion: "neither writes a tracked record".

**4. LOW: the judge-guard revert cannot undo an added untracked path.**
- Changed text: D9, "whose way forward is the parent reverting them in the run's worktree, for example with `git checkout <head> -- <path>`, then `lym loom resume`".
- Summary: the snapshot covers untracked paths. A file the judge created is not removed by `git checkout <head> -- <path>`, and a weft path needs the weft HEAD, not the warp's. The named step does not reach a tree that matches the snapshot for those cases.
- Evidence: the snapshot "snapshots each checkout's HEAD, tracked tree and untracked paths, warp and weft alike", and "A resumed row compares against the recorded snapshot … until the trees match it".

**5. LOW: the moved-warp way forward names no verb.**
- Changed text: D6, "the parent resetting the run's warp branch to the baseline head followed by `lym loom resume`".
- Summary: the record names no verb or guard for the parent to rewrite the run's warp branch from its own worktree. This alternative may not be reachable in practice, though `defer` still is.
- Evidence: the support log's round-1 line (existing-paths 3) found the parent cannot reach the run's worktree for `amend`, and fixed that with a slug-taking verb. This step names no equivalent.

The other ways forward are reachable and make progress:
- a stored `does not match` names amend and the code-fixing route;
- an exhausted `judgement` entry leads to a respawn;
- an interrupted judge is probed, then resumed or respawned;
- `defer` is admitted at the widened rows;
- a waived run is Done;
- an old agent is ended through `EndRole`.

Tool calls:
- Read `internal/driftengine/verbs.go` `Defer` (lines 226–242), to check how far `defer`'s current admission reaches. My context did not show it.
