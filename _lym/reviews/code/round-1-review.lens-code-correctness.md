```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes through one concrete input and checked the result.

**Cases run**
- `PRReview.Call` → `decide`: review mode, no record, open/merged status. Correct.
- `closeRecordedPullRequest`: decision no, recorded PR open. Closes it and drops the record; no session to stop. Correct.
- `openPullRequest`, re-entry after the gate-failure halt (record GatedHead=h1, task head h2 that the gate refused): `pushTask` at prreview.go:171 runs before any gate, so h2 reaches the PR branch. **Wrong** (F1).
- `openPullRequest`, re-entry after a foreign-head halt where the operator brought the hand commits in (head h3): h3 is pushed ungated before `wake` gates it. **Wrong** (same cause, F1).
- `wake` / `gateHead`, gate fails on h2: halts, pushes nothing, GatedHead stays h1. Correct on its own; the resume undoes it (F1).
- `gateHead`, runner crash: halts with the crash reason. Correct.
- `gateHead`, head moved or tree dirtied during the gate: verdict discarded, wait goes on. Correct.
- `wake`, hand commit X on GitHub plus a new task commit h2, both on h1: the forced poll returns open at X, then `gateHead` passes on h2, then the non-force `PushRun` is rejected as non-fast-forward. `pushTask` halts with `pushHaltReason` ("run lym fabric push…"), and `foreignHeadHalt` never runs. **Misreported** (F2).
- `foreignHeadHalt`, status HeadSHA empty, equal to GatedHead, or equal to head: wait goes on. Correct.
- `foreignHeadHalt`, GitHub head an ancestor of head (lag): `CommitsMissing(head, x8)` returns none, wait goes on. Correct.
- `foreignHeadHalt`, CommitsMissing returns an error: halts as foreign, naming the error. Correct by its doc.
- `judge`, open PR with Continue set and a foreign head: the foreign halt comes first. Correct.
- `judge`, open PR approved at GatedHead == head, clean tree: Done. Correct.
- `judge`, closed PR, Continue set, clean, gated: `endReview`. Correct.
- `judge`, closed PR, no Continue: halts. Correct.
- `judge`, merged PR, every branch: unchanged and correct.
- `holdWhileCovered`, covered then uncovered: returns; no session work left. Correct.
- `landingForDecision`, review mode, closed PR, ReviewEnded false: halts. Correct.
- `landingForDecision`, review mode, closed PR, ReviewEnded true: reopens and merges. Correct.
- `landingForDecision`, trail mode, closed PR: reopens. Correct. Its doc comment is stale (F5).
- `prContinueVerb`, strand set: refused before any read. Correct.
- `prContinueVerb`, strand empty, no Continue: writes Continue. Correct.
- `prContinueVerb`, Continue or ReviewEnded already set: no write. Correct.
- `prContinueVerb`, absent or undecodable record: refused or error. Correct.
- `AnswerParentAsk`, ended ask (no-parent or undelivered): refused. Open ask past its deadline: set timeout. Correct.
- `rewriteHubFile` with `renamedModes`, `pull_request: on` (unquoted, parsed as !!str): rewritten to "review", no required flag. Correct.
- `rewriteHubFile`, empty or non-mapping file: returned unchanged. Correct.
- `ReinterpretPullRequestModes`, a worktree's local file holds merge: flagged with `--set pull_request=trail`. Correct.
- `ReinterpretPullRequestModes`, rewrite text for an on in a hub whose `beam.autonomy` is true: claims "which keeps what it meant". **Misreported** (F6).
- `ReinterpretPerTaskSettings`, required: becomes review and is flagged; the pull-request-files line moved into the caller. Correct.
- `CarryBeamTaskSettings`, entry run_settings holding merge on a machine that has never carried: carried as trail. Correct.
- `CarryBeamTaskSettings`, a machine that already carried before this change: the marker keyed on `beam-per-task-settings` already exists, so the carry never runs again. Board entries' run_settings holding on or merge are never rewritten, and the new `CheckLocalValues` refuses them at seed. **Wrong** (F3).
- `agentconfig.Load`, hub agents.yaml holding `skills.pr-reviewer` or `efforts.pr-reviewer/&lt;model&gt;`: `skillLists` / `checkEfforts` now refuse the whole file. **Wrong** (F4).
- `ValidateLocal`, local landing file with `pr_reviewer`: retired with CategoryNone, admitted. Correct.
- `ValidateMachine`, machine landing file with `pr_reviewer`: refused, since the retired category is not CategoryMachine. Behaviour is unchanged, but the new comment claims otherwise (F7).
- `ValidateLocal`, loom local file with `pr: {review_small_max: 1}` or a flat `pr_review_*` name: retired with CategoryNone, admitted. Correct.
- `AutonomyForcedSetting("pr.review_ask_parent")`: no longer forced. Correct.
- `reviewInputs`, no fixer report, or a fixer report beside a pr-findings file: answers none, or the report only. Correct.
- `startNextGeneration` / `reenterUnchanged` without `hasFullInput`, kept input with FullGeneration off: amendment. Correct.
- `gotoAdmitted`, nil location: returns nil. Otherwise it retires interrupted runs and returns nil. Correct.
- `PRReview` with a run in flight at upgrade whose record names a live session: the row no longer reads StrandGUID, so the old session is never stopped. **Inconsistent state** (F8).

**Findings**

F1: BLOCKING, design. `internal/landingshed/prreview.go:171`, `openPullRequest`. The resume after a gate-failure halt, or after a foreign-head halt once the commits are brought in, runs `pushTask` before the wait gates anything. A head the pr-review gate refused, or never ran, reaches the pull request branch. This contradicts the `gateHead` doc ("A failure pushes nothing; the resume gates the task head again", prreview.go:516-517) and the halt text at :539 ("run lym loom resume, which gates the task head again"). It also contradicts the foreign-head halt text at :689 ("which gates the task head and pushes it"). Evidence: prreview.go:171-173 run the push before `wait` (:244). `gateDue` is set only inside `wake` (:473).

F2: MEDIUM, design. `internal/landingshed/prreview.go:473-481` (`wake`). The gate and its push run before `judge` reaches `foreignHeadHalt`. Case: a hand commit on GitHub coincides with a new task-worktree commit. The gate passes, the non-force push is rejected, and the row halts with `pushHaltReason` ("run lym fabric push in the task worktree…", finalize_pr.go:57-58). A fabric push fails the same way. The foreign-head halt, the one way forward that applies, is never reached. Evidence: prreview.go:479-480 call `gateHead` before `judge` at :498; `gateHead` pushes at :541; `PushTask` is a plain `pairteardown.PushRun` (landingdeps.go:409-416).

F3: MEDIUM, design. `internal/dobbyengine/reinterpret.go` (`CarryBeamTaskSettings` rename mapping) and `internal/hubreconcile/reinterpret.go:119`. Board entries whose `run_settings.beam.pull_request` is on or merge are mapped only by the carry, and the carry runs once per machine behind the old `beam-per-task-settings` marker. On any machine that has already carried, nothing rewrites them. Seeding such a task then runs `checkLocalDocument` → `CheckValues` (configreg/runsettings.go:219-225), which now refuses on and merge (dobbyengine/config_test.go new rows). The new reinterpretation never touches the board.

F4: MEDIUM, design. `internal/agentconfig/config.go:47-52`, the pr-reviewer role removed with no reinterpretation. If a hub's agents.yaml in any layer sets `skills.pr-reviewer` or an efforts key `pr-reviewer/&lt;model&gt;`, `Load` refuses the whole file (config.go:164-166 and :219-221). Every role whose start reads agents.yaml then fails until the operator removes the key by hand. The repo's earlier role retirement shipped a reinterpretation for this: the `_bolt/_lym/config/reinterpreted-agents-treadle-roles-retired` marker.

F5: LOW, consistency. `internal/landingshed/finalize_pr.go:327-328`. The doc comment of `landingForDecision` still says "A closed pull request whose record holds autonomy true halts whatever the operator's ending and MergeAtOnce say. It is neither reopened nor merged." The diff removed that branch (:357), so the comment misstates the behaviour.

F6: LOW, consistency. `internal/dobbyengine/reinterpret.go`, `retiredMeaning`. The rewrite and flag of on → review say "which keeps what it meant". Under `beam.autonomy: true`, on meant that the parent's satisfied answer ended the review; review now waits for the operator whatever the autonomy. For an autonomous hub the text states a meaning it no longer keeps. Evidence: the `retiredMeaning` case returning "which keeps what it meant"; the old flow in the removed `judgeParentAnswer` (diff 020, prreview.go).

F7: LOW, consistency. `internal/landingshed/config.go`, the new `RetiredKeys` comment. It says the local and machine layers "refuse all but landing.pr_reviewer". `ValidateMachine` admits a retired key only when its category is CategoryMachine (configengine/layers.go:357-360), so the machine layer refuses `landing.pr_reviewer` too.

F8: LOW, design. `internal/landingshed/prreview.go` (the session stop was removed from `Call`, `closeRecordedPullRequest`, `wake` and `holdWhileCovered`). A run waiting at PR-Review when the binary is upgraded keeps its live pr-reviewer session, and the row no longer reads `StrandGUID` to stop it. The session runs until teardown, and its prompt still directs it to `lym loom pr-findings`, which is gone, and `pr-continue`, which now refuses it. Evidence: the removed `endSession` / `stopLiveSession` (diff 024, prsession.go); `Record` drops the strand fields (pullrequest/record.go).

Tool calls:
1. `pushHaltReason`, the `rewriteHubFile` body, and `CheckValues` and run_settings sites in configreg, hubreconcile and boardengine. My context lacked the push halt text, the hub rewrite body, and whether anything rewrites board run_settings.
2. `internal/configreg/runsettings.go:195-235`. My context lacked how seed-time run_settings values are checked.
