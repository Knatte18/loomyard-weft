```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: tracing whether each guard, refusal, branch and test row the cards add can be reached.

**Per guard: reachable or not**
- Card 1, `CommittedFileDiffs`/`CommitMessages` unresolvable-ref errors: reachable, by the test rows with a bad base or `from`.
- Card 2, `filesFindings` refusals (unknown or duplicate id, unruled file, bad ruling, empty reason, `none` with checks, `loosened` with none, empty check field, `caused` with no cause): reachable through `NewJudgementGate` → `ParseJudgement` once `JudgeRequest.Files` fills `want.FileIDs` (judge.go:116, judge.go:210).
- Card 3, uncaused halt in `checkAndJudge`: reachable. A failed check leaves `LastPassHead` unset, so the early Done at verify.go:158 does not hide it. A trailer-only commit moves HEAD, which changes the key and respawns the judge.
- Card 3, uncaused halt in `passCommand` on compare exit 0: reachable (verify.go:818 is the site it now precedes).
- Card 3, the "both reasons" row (cell does not match plus file uncaused in one spawn): reachable.
- Card 3, nil `ReviewBase` lists no file: reachable for a standalone run (wiring.go:774–776 leaves `verifyMergeBase` nil).
- Card 3, `driftVerifyEntry` requiring `ChangedFiles` and `CommitMessages`: a fail-loud wiring guard that envkit fills.
- Card 6, missing plain fragment error: reachable by the test row only. Production seeding writes a new fragment, so no run hits it. Acceptable for a wiring-style guard.
- Card 7, a full input re-opening an unchanged subject, with and without `FullGeneration`, and the round-0 settle row: reachable. The verb writes the copy after the settle, so its mtime is later than `settledAt` (amendment.go:73–74). An older `pr-findings-&lt;n&gt;` falls before the next settle and is not kept.
- Card 8, step 1, dirty worktree: reachable, but the verb's own input file trips it. See finding G1.
- Card 8, step 2, unparsable file: reachable through `ParseReview` (verdict.go:111).
- Card 8, step 3, no pull-request record: reachable by a direct CLI call before PR-Review opens the record, as for `pr-continue` (prcontinue.go:24).
- Card 8, step 4, failed `smallMax` or `edge` read: reachable through the fakes.
- Card 8, step 5, record already marked: reachable between the verb's write and the persisted reroute.
- Card 8, `sendBackEdge`'s three false reasons:
  - no `OnReroute`: reachable in tests over `ProducerDef`.
  - target `EntryGate` false: reachable in sampler (`entry_gate: sampler.code_review`, sampler-recipe.yaml:123).
  - target `MaxBounces` 0: reachable, since `applyReviewBudget` sets the Code-Review budget onto Code-Bouncer (loomrecipe.go:199).
- Card 8, the `commit` failure re-run (`CountedSets` dedupe, step 5 re-answer): reachable.
- Card 9, `checkWiring` requiring `ShedStatus`: a wiring guard. The only production builder is `pullRequestDepsGuardedBy` (landingdeps.go:369); the test builders are prreview_test.go:667 and envkit.go:239, which the card lists. finalize_pr_test.go:489 builds deps for Finalize, which does not run PR-Review's `checkWiring`.
- Card 9, `settleSendBack`:
  - newest own entry the marked file's reroute: reachable, since reroute.go:59 writes `Output: output.Path`.
  - first-send-back crash, with no own entry: reachable.
  - reroute of another file (crash after a later send-back's verdict, before its persist): reachable. After the clear the row's newest own entry stays the earlier reroute until its next verdict persists.
  - `RerouteCapped` Stuck: reachable, since `rerouteCount` stops at it (reroute.go:23).
- Card 9, dirty-tree Stuck in `sendBack` (wake and Call): reachable, but its common trigger is the verb's own input file. See G1.
- Card 9, wake's `sendBack` before any gate: reachable, because the verb's record write fires `RecordChanges`.
- Card 10, the edge: reachable only through a marked send-back. Only the three recipes it edits run `engine: PRReview`.

**Findings**

G1 — MEDIUM, class design, location `_lym/plan/08-loom-pr-findings-verb.md` (with `_lym/plan/12-result-check-wording.md` and `_lym/plan/09-pr-review-send-back-row.md`).
- **Summary:** neither card 8 nor card 12 says where the PR-Review agent writes the findings file. Untracked files count as dirty, so a file written in the task worktree makes the verb's own dirty-worktree refusal (step 1), and the row's dirty-tree Stuck, fire on the verb's input.
- **Why it matters:** both ways forward then ask the agent to "commit or discard" the paths. Committing lands the findings file on the pull-request branch; discarding deletes the input before the verb can read it.
- **Evidence:** `internal/verifytree/verify.go:274-280`. `DirtyPaths` returns every `git status --porcelain` path, untracked included, ignored excluded. Card 8 wires `dirty` with `verifytree.DirtyPaths`. Card 12's brief text says only "writing them first in burler's review format", with no location.
- **Fix:** card 12's PR-Review brief, and card 8's `--help` text, name a location outside the tracked tree for the findings file, such as the worktree's git-ignored `.scratch/`. Card 8 adds a row to `TestPRFindingsVerb`: a findings file under the ignored scratch path passes the dirty check.

Tool calls:
- `internal/verifytree/verify.go` `DirtyPaths`, plus a grep for `PullRequestDeps{` construction sites: the manifest named `DirtyPaths` but never showed whether it counts untracked files, and it did not list every builder of the deps that card 9's new `checkWiring` field must cover.
- A grep of the recipes for `engine: PRReview` and of the PR-Review brief for a findings-file location: to confirm which recipes `sendBackEdge` can face and whether the brief names a location. The manifest held only the three recipes' PR-Review rows and no search of the brief for a location.
