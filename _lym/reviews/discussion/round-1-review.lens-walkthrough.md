```yaml
lens: walkthrough
outcome: reported
```

**Decisions walked:**

- **D1** — Case: a Webster fork widens `tol 1e-6→1e-3` in an existing test and adds `Loosened-Check: TestX — tol — measured 4e-4 after solver change` on a later commit. The reviewer already reads `git log` over the range (`loom-rubric-code-review.md:80`) and judges the trailer. **Purpose met.**
- **D2** — Case: a fork forgets the trailer; Code-Burler-Fix adds an empty commit carrying it. Trailer sits on a range commit. **Purpose met.**
- **D3** — Case: widened limit in an existing test, no trailer → Drift-Verify Stuck `uncaused` → parent adds an empty trailer commit → `lym loom resume`. **Purpose missed** (finding 1).
- **D4** — Case: the parent's creel review carries 5 MEDIUM findings; the agent writes the file and runs the verb on a clean tree → send-back → PR-Review reroutes → Code-Bouncer re-entry → … → PR-Review. **Purpose missed** (findings 2, 3, 4).
- **D5** — Case: `run_settings` sets `pr_review_small_max: 5`; the verb reads it like `pr_review_ask_parent` over the layers (`internal/loomcli/landingdeps.go:446`). **Purpose met.**
- **D6** — Case: first send-back writes a note via `friction.NotePath` (non-clobbering). **Purpose met**, but the note names a stored findings file that the next generation's archive moves (finding 2).
- **D7** — Case: Describe rewrites the description after Code-Review and keeps out a figureless claim. **Purpose met.**

**Findings:**

1. **BLOCKING, design** — `_lym/discussion/decision-record.md` D3, lines 67 and 70–71.
   Summary: Following D3's own way forward never clears an `uncaused` Stuck. Rulings are keyed only by a hash of the file's diff; an empty commit carrying the trailer leaves that diff unchanged, so the resumed Drift-Verify reuses the stored `uncaused` ruling and halts again. Only the revert path changes the hash. No acceptance criterion checks that the trailer way forward un-sticks the row.
   Evidence: decision-record.md:67 ("keyed by a hash of the file's diff, and reuses it while that diff is unchanged"), :71 (way forward: "a commit carrying the trailer … then `lym loom resume`"); `internal/driftengine/verify.go:287` already halts on a reused stored ruling that does not match.
   Fix: include the range's `Loosened-Check:` trailers naming the file in the key (or re-rule `uncaused` entries whenever trailers change), and add a criterion: trailer commit → resume → Done.

2. **BLOCKING, design** — `decision-record.md` D4, lines 92 and 101–102; D6, line 124.
   Summary: The send-back file is stored in `_lym/reviews/code/`, which is the Code-Bouncer's run directory (RunRoot = `LoomReviewsDir`, `run_subdir: code`). Starting the new generation renames that directory into an archive sibling and carries back only the settle record, settle patch and generation record. So the review input path handed to the reviewers, and the path D6's friction note names, point at a file that is gone. The Parent-Align fixer report survives because it lives in the scratch directory.
   Evidence: `internal/loomcli/wiring.go:1181` (RunRoot), `contracts/recipes/darn-recipe.yaml` Code-Bouncer `run_subdir: code`, `internal/shedadapters/archive.go:67-76`, `internal/shedadapters/settlerecord.go:269`, `internal/loomcli/subjects.go:283` (fixer report in scratch dir).
   Fix: store send-back files outside the run directory, e.g. beside the fixer report in the loom scratch directory.

3. **MEDIUM, design** — `decision-record.md` D4, lines 102 and 111; D6, line 125.
   Summary: "A PR-findings input starts a full generation" while the review input names the send-back file, but in today's code a full generation drops the review input: `keptReviewInput` is read only for an amendment, and only an amendment record carries `ReviewInput`. The reviewer learns of an input only through the amendment note; the rubric's `## Review input` opens "When the amendment note names a review input", and a full generation renders the amendment marker as `none`. So the Rationale's "the Bouncer's review input and full generation exist, so only the mark, the verb and the second input source are new" is false: a full generation that carries an input, and a channel naming it to the reviewer outside the amendment note, are both new. Also, with `FullGeneration` set, an unchanged subject resettles with no review, skipping the input check, and a send-back with no new commits is the normal case, since the brief says run the verb before fixing. That path would approve immediately.
   Evidence: `internal/shedadapters/amendment.go:31-37, 46-51`, `internal/shedadapters/bouncer.go:741-748`, `contracts/stencils/loom/loom-rubric-code-review.md:137-141`.
   Fix: state that the send-back input forces a new generation on an unchanged subject and is named to the reviewer outside the amendment note; correct the Rationale.

4. **MEDIUM, design** — `decision-record.md` D4, line 103.
   Summary: "PR-Review again, which finds the open pull request and, with `pr_review_ask_parent` on, asks the parent again" is false against the code: `askParentReview` returns early when `rec.ParentReviewAsked` is set, and the re-entry keeps the existing record (`recordPullRequest` with `hadRecord` only updates PR fields). So the parent is not asked again, and the re-entered row waits on a review nobody was asked for. D4 states the re-ask as existing behaviour, so a plan writer won't add the needed reset.
   Evidence: `internal/landingshed/prreview.go:345`, `:445-456`.
   Fix: D4 states that the row clears `ParentReviewAsked` when it clears the send-back mark; add a criterion that the re-entered PR-Review asks the parent again.

Tool calls:
1. `internal/shedadapters/amendment.go:1-60` — to see whether a full generation keeps the review input; context had only grep lines.
2. `archiveRunDir` + `ensureSession` — to see what a new generation does to the run dir, and how the PR-Review session re-enters.
3. Grep `CarryRecords` / `run_subdir` — to learn which files survive the archive (wrong file in first grep).
4. `CarryRecords` and loom RunRoot — to confirm the carried files and that `_lym/reviews/code` is the run dir.
