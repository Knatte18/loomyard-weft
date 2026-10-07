---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/gitrepo/doc.go:67
    summary: New and changed Go comments in gitrepo/doc.go, webstercli/cli_test.go and websterengine/beginbatch_test.go are fixed-column wrapped, with two sentences on one line, against PATTERN-comment-line-breaks
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/websterengine/doc.go:250
    summary: The package doc still says begin-batch idempotently asserts Master's model, contradicting the new "one model per run" section and card 4's removal
  - id: F3
    severity: MEDIUM
    class: consistency
    location: contracts/stencils/loom/loom-rubric-plan-review.md:31
    summary: The plan-review rubric stencil still pins twenty-eight check IDs ending at commit-subject-mismatch, but the diff makes the spec's validation-checks section list 31 rows ending at delete-target-gone
  - id: F4
    severity: LOW
    class: design
    location: internal/landingshed/publish.go:262
    summary: A push rejected with zero remote-only commits (a remote hook refusal, or no remote branch) still gets the merge-then-resume way forward, which cannot clear it and loops to the same stop
---

Range: `_lyx/shed/bugfix-webster/status.json` gives `product.parent` = `main`, and no `_lyx/loom/rework/` round exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD`, twelve commits, one per card.
`go build ./...`, `go vet ./...`, `gofmt -l` over the changed files, `go test ./internal/... ./cmd/... ./contracts/... ./tools/...` and the plan's `-tags integration` set all pass.

Verdicts on the focus questions, each grounded in code and tests I read:

- Cards 1–3 and 7, D1–D4 scope: `ValidateDispatch` runs `LaterDeleteReferences` over the pending plan only (`resolvePass`'s `current`), and `downgradeGoneDeleteTargets` runs only when `len(done) > 0`, over `pendingCardsByID`.
  `Validate`, `ValidateFormat` and `ValidateRework` never reach the downgrade, so a whole-plan entry point still refuses a missing Delete target; `TestValidateDispatch_DeleteTargetGone` pins the no-completed-card row, and the Uses and own-Edit rows keep the blocking finding.
  Not a defect.
- Card 7, `delete-not-done` caused by a later card: `failCardNotDone` adds the `LaterDeleteReferences` reasons over `unbegunCards` and swaps the way forward to the plan edit, or to the `--fresh` steps for an uncheckable record. `RecoverSpawnOrAttach` refuses with `ErrRecoveryDeleteReferenced` before any spawn, and the CLI maps it to `batch_failed`.
  The loop is closed for the plan-edit path.
  Not a defect, apart from F4's separate Publish loop.
- Cards 5 and 6: `BeginBatch` archives only when `State.Batches[number]` is nil. `Rebaseline` collects restamps and applies them after every refusal check, and never saves; the CLI saves only on success.
  A failed `restampBaseline` after the in-memory restamp returns an error and the CLI discards `st`, so no state is written wrongly.
  Not a defect.
- Card 4: no reader of `AssertedModel`, `Injector`, `ModelSwitchSequence` or the begin-batch `model` envelope key remains in code, stencils, specs or the sandbox suite; the one remaining prose trace is F2.
- Cards 9–11: `moveRemoteTaskBranch` refuses when `RemoteOnlyCommits` lists any commit HEAD lacks, skips when the remote tip is at or behind `sha`, and pushes under `--force-with-lease` on the tip it read.
  The remote therefore never moves past commits HEAD lacks, and a moved tip is `ErrLeaseRejected`.
  Not a defect.
- Card 12: the reason names the tip and `len(commits)` exactly as `RemoteOnlyCommits` returns them, and the landingshed integration test resumes to the GitHub step; the one gap is F4.
- Per-card mechanical checks: the plan's cards carry only Create and Edit groups, so no Rename or Delete mechanical check applies, and each Create target (`deleteorder.go`, `remoteonly.go`, the new test files and the three gitrepo symbols) exists in the diff.
- Test economy: I found no redundancy I can ground by naming a covering test; the engine and cobra-level tests for one behavior each pin different rows or the plan-mandated seam.
- Docs: `doc.go` for `planglyph`, `fabricengine`, `gitrepo`, `landingshed` and `websterengine`, `refusal-spec.md`, `loom-plan-spec.md` and the Master stencil changed with the code; F2 and F3 are the gaps.

### [BLOCKING] Fixed-column wrapped comments in changed Go comments

**Location:** `internal/gitrepo/doc.go:67-72`, `internal/webstercli/cli_test.go:381-383`, `internal/websterengine/beginbatch_test.go:3-4`
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line in new and changed Go comments.
The new `gitrepo/doc.go` bullets break mid-sentence at a column and put two sentences on one line ("into failure. CommitsNotIn is its listing sibling: ...").
The rewritten `cli_test.go` comment lines put "informational in both scopes. A third card Deletes ..." on one line and wrap mid-sentence.
The rewritten `beginbatch_test.go` header lines put "seam is a shuttlefake.Reed. The plan" on one line.
**Fix:** Rewrite only these new or changed lines with one sentence per line, breaking further at independent-clause boundaries, and leave untouched legacy lines alone.

### [MEDIUM] Stale model-assertion sentence in the websterengine package doc

**Location:** `internal/websterengine/doc.go:249-251`
**Issue:** The "bracket verbs, not spawn/poll" section still says begin-batch "idempotently asserts Master's model for this batch".
Card 4 removed that behavior and rewrote the later section to "one model per run", which says begin-batch never reads or changes the model, so the two sections of one document disagree.
**Fix:** Drop "idempotently asserts Master's model for this batch," from that sentence, keeping the pause and fingerprint checks, the start-SHA record and the prompt render and write.

### [MEDIUM] Plan-review rubric stencil pins a stale check count and range

**Location:** `contracts/stencils/loom/loom-rubric-plan-review.md:17,20,31`
**Issue:** The stencil says the plan spec lists "twenty-eight check IDs ... `format-unrecognized` through `commit-subject-mismatch`", and "twenty-seven of the twenty-eight" run in the format-only gate.
The diff's `loom-plan-spec.md` now numbers 31 rows, inserts `uses-later-target` at 23, and ends with `delete-before-reference` and `delete-target-gone`, which `internal/planglyph` emits rather than the format-only gate.
The diff itself replaced the same kind of tally in `planparser` comments and the spec's `LICENSE` sentence, and left this one, which an LLM reviewer reads as the definition of what the plan gate already checks.
**Fix:** Rewrite the three sentences without a count or a named last row, for example "every check ID the spec's validation-checks section lists is enforced deterministically, all but `plan-unapproved` by this round's own gate over this round's own output", and name the planglyph-emitted rows only if the distinction matters to the reviewer.

### [LOW] Rejected push with no remote-only commits still prescribes a merge that cannot clear it

**Location:** `internal/landingshed/publish.go:262`
**Issue:** `ErrPushRejected` matches any push stderr containing "rejected", including `! [remote rejected] ... (pre-receive hook declined)`, and the new refusal-spec row lists "a remote hook refuses it" as a cause.
For that cause, or an absent remote branch, `RemoteOnlyCommits` returns zero commits, so the reason reads "the remote task branch is at <tip> and holds 0 commit(s) the local branch lacks; way forward: run `git merge origin/<task-branch>`" and, for an absent branch, shows an empty tip.
The merge reports "Already up to date", `lyx loom start` re-runs Publish, and the same rejection stops the run again, which is the retry loop card 12 exists to remove.
**Fix:** When the read succeeds and returns no commits, state that the remote holds nothing the local branch lacks, so the remote rejected the push for its own reason, and name that remote rule as the thing to clear before `lyx loom start`; keep the merge way forward for a non-zero count.
Add the zero-commit case as a row of `TestPublish_StuckBeforePullRequest` and align the `Publish: push rejected` row's trigger and way forward in `refusal-spec.md`.
