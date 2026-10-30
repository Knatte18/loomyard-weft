---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/11-pr-review-row-opens.md:8
    summary: no card edits landingshed's import allowlist, so card 11's own tier0 gate fails once landingshed imports internal/pullrequest and internal/githubclient
    fix: "In card 11 add `internal/landingshed#landingshedAllowedImports` and `internal/landingshed#TestToldGeometryInvariant_AllowlistOnly` under **Edit:**, and state in its Intent that the allowlist gains `github.com/Knatte18/loomyard/internal/pullrequest` and `github.com/Knatte18/loomyard/internal/githubclient`, which the row's production code imports for the `API` seam type and for `errors.Is(err, githubclient.ErrTokenUnresolvable)`. In card 14 add the same two **Edit:** targets and state that the allowlist gains `internal/parentdirective` and `internal/editdirective`, which `ensureSession` imports to render the two directives. Checked with `lym loom validate-plan --overlay`: the combined run reports no blocking finding the baseline lacks."
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lym/plan/16-teardown-closes-pull-request.md:40
    summary: card 16's new guard bans internal/pullrequest imports outside landingshed, pairteardown and loomcli, yet card 11's envkit LandingDeps stub must import it, so the guard fails on envkit.go
    fix: "In card 16's Intent, change the third clause of `TestPullRequestGuard_OnlyLandingAndTeardown` to fail on a file outside `internal/landingshed`, `internal/pairteardown`, `internal/loomcli` and `internal/testkit/envkit` that imports `internal/pullrequest`, and say envkit is allowed because its `LandingDeps` stub names `pullrequest.API` in the `API` seam's func literal. Text-only change, no target moves."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/11-pr-review-row-opens.md:90
    summary: README.md and loomrecipe/doc.go still call PR-Review a stub that always ends Done after card 11, and no card lists the README or rewrites that doc sentence
    fix: "In card 11 add `README.md` under **Edit:** and state in its Intent that the Landing bullet's sentence `PR-Review is a stub for a human pull-request review that is not built` is rewritten to say PR-Review lands the task through a GitHub pull request the operator discusses with an agent when the resolver says so. State also that the sentence `PR-Review always ends Done, so it carries no on_stuck` in `internal/loomrecipe/doc.go` (already an **Edit:** target) is rewritten to say its Stuck is always a Halt, so it carries no on_stuck. Checked with `lym loom validate-plan --overlay`: no new blocking finding."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lym/plan/13-pr-review-row-waits.md:63
    summary: wait's closed-with-ClosedForContinue Done verdict is unreachable because card 11 step 6 deletes a ClosedForContinue record before every wait, which also drops the operator's ending after a crash between the record write and Done
    fix: "In card 11 step 6, remove a `ClosedForContinue` record and open afresh only when `TaskHead()` differs from the record's `GatedHead`, the Finalize-reroute case whose new commits need review; when the head equals `GatedHead`, keep the record and let `wait` end Done on its closed-and-`ClosedForContinue` verdict, so a re-entry after a crash between `ClosedForContinue` and Done keeps the operator's ending. Add to `TestPRReview_Call` a row for each case (equal head keeps the record and ends Done without creating a pull request; moved head removes it and creates one), and make `TestPRReview_Wait`'s `ClosedForContinue` row start from a record that `Call` hands over."
  - id: F5
    severity: MEDIUM
    class: design
    location: _lym/plan/13-pr-review-row-waits.md:56
    summary: card 13 mails the agent the reason of verifyGate.check and says it names the failing tier, but check's reason names no tier and its wording (landing gate, after merging parent branch, then resume) is wrong for an agent mail
    fix: "In card 13 add `plan:internal/landingshed#verifyGate.checkHead` -> `func (g verifyGate) checkHead(ctx context.Context, producer string) (string, error)` under **Create:** and `internal/gateslot#FailedTier` under **Uses:**. State in the Intent that `wait` calls `checkHead` in place of `check`: it runs the key's call through `g.verify` with the same wait marks, returns an empty string on a pass or a skip, and on a failed result returns `the pr-review gate failed at tier <n>; output: <log>; fix it in a new commit` with `<n>` from `gateslot.FailedTier` over the log, else the highest tier `g.tiers(g.key)` lists; a dirty result returns the dirty reason, which `wait` discards since its after-gate read sees the dirty tree. Make `TestPRReview_Wait`'s failing-gate row assert that the mail names the tier and the log path and holds neither `merging parent branch` nor `resume`. Checked with `lym loom validate-plan --overlay`: no new blocking finding."
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/13-pr-review-row-waits.md:57
    summary: wake step 4 adds an immediate Status poll when Continue first appears, a second poll inside one minute that D9's at-most-once-a-minute bound does not allow, and no Shared Decision records the departure
    fix: "Add a bullet to 00-overview.md's Shared Decisions recording that one poll per run may follow the continue flag at once, bounded to the first wake that reads `Continue`, so the operator's ending is not held up a minute, and that every other poll keeps the once-a-minute bound. In card 13's `TestPRReview_Wait`, add to the row where a fake writes `Continue` between two wakes an assertion that the `Status` call count rises by exactly one and stays at most one per minute on later wakes."
---

### [BLOCKING] Landingshed's import allowlist is edited by no card

**Location:** `_lym/plan/11-pr-review-row-opens.md:8`
**Issue:** `internal/landingshed/seam_enforcement_test.go` runs `scankit.AssertImportAllowlistNoStale` over the package's production files, so an import outside `landingshedAllowedImports` fails `TestToldGeometryInvariant_AllowlistOnly`, a tier0 test in the card's own package.
The list holds neither `internal/pullrequest` nor `internal/githubclient`.
Card 11 makes `PullRequestDeps.API` return `pullrequest.API` and judges failures with `errors.Is(err, githubclient.ErrTokenUnresolvable)`, so its production code imports both.
Card 14 renders `internal/parentdirective` and `internal/editdirective` in `ensureSession`, and landingshed imports neither today, since only `internal/mergeresolve` does.
No card names the allowlist or its test; a grep of the plan for `allowlist`, `AllowedImports` and `seam_enforcement` finds only card 3's unrelated phrase.
`loomshed`'s check `seam-test-unlisted` does not see this, since the file marks no struct, so the card gate for card 11 is where it first fails.

### [BLOCKING] Card 16's guard rejects the envkit stub card 11 requires

**Location:** `_lym/plan/16-teardown-closes-pull-request.md:40`
**Issue:** `TestPullRequestGuard_OnlyLandingAndTeardown` walks every non-test `.go` file with `scankit.Walk`, which skips only `.git`, `_lym`, `.lym`, `_mill`, `.scratch`, `.wiki`, `_raddle` and `testdata` directories, so `internal/testkit/envkit/envkit.go` is walked.
Card 11 has envkit's `LandingDeps` fill `PullRequest.API`, whose type is `func(originURL string) (pullrequest.API, error)`; a non-nil func literal of that type names `pullrequest.API`, so envkit imports `internal/pullrequest`.
The guard's third clause fails every importer outside `internal/landingshed`, `internal/pairteardown` and `internal/loomcli`, so card 16's own tier0 gate over `cmd/lym` fails on envkit.
`internal/testkit/envkit/envkit_test.go`'s `TestFullEnv_FillsEverySeam` needs the seam non-nil, so the stub cannot drop the field.

### [MEDIUM] README.md and one loomrecipe doc sentence are falsified and unlisted

**Location:** `_lym/plan/11-pr-review-row-opens.md:90`
**Issue:** `README.md:70` says `PR-Review` is a stub for a human pull-request review that is not built, and `internal/loomrecipe/doc.go:16` says PR-Review always ends Done, so it carries no on_stuck.
Card 11 makes both false.
It rewrites the recipe header comments, `docs/overview.md` and the pawl sequence in `internal/loomrecipe/doc.go`, but no card lists `README.md`, and card 11's Intent does not name the loomrecipe doc sentence, whose rewrite is not covered by its `Every header comment` sentence since that sentence is about recipe YAML headers.
CLAUDE.md's `Docs land in the same commit` rule covers both.

### [MEDIUM] The closed-with-ClosedForContinue verdict can never run

**Location:** `_lym/plan/13-pr-review-row-waits.md:63`
**Issue:** `wait` is entered only from `Call`, and `Call`'s step 6 (card 11) removes any record marked `ClosedForContinue` before it, opening a fresh pull request.
Inside one `wait`, the only writer of `ClosedForContinue` is the open-with-`Continue` verdict, which returns Done in the same wake, so the verdict `closed and not merged: Done when the record carries ClosedForContinue` is dead code, and a `TestPRReview_Wait` row for it, which the card plans through `Call`, cannot reach it.
The dead verdict was written to survive a crash between recording `ClosedForContinue` and returning Done, and step 6 defeats that: the re-entry deletes the record and opens a second pull request, so the operator's ending is lost and the run waits again.
Finalize's matching rule (card 15, `ClosedForContinue` with a closed pull request lands locally) is reachable and unaffected.

### [MEDIUM] The gate mail's content rests on a false premise about verifyGate.check

**Location:** `_lym/plan/13-pr-review-row-waits.md:56`
**Issue:** Card 13 posts to the agent `the gate's reason, which names the failing tier and log path`, as D8 requires.
`verifyGate.check` returns the Stuck reason built by `verdict`, which reads `landing gate failed after merging parent branch "<branch>" (exit code N); output: <log>; fix forward on the task branch, then resume`.
It names the log path but no tier, since the tier is derived only inside `recordFailure` for the failure record, which the new gate writes none of (nil recorder).
The wording also misleads an agent in a live session: the pr-review gate follows no merge, and `resume` is the operator's step, not the agent's.
The planned `TestPRReview_Wait` failing-gate row would pass on a fake whose reason is invented, leaving D8's `naming the failing tier` unmet.

### [LOW] The immediate poll on Continue departs from D9's poll bound without a recorded decision

**Location:** `_lym/plan/13-pr-review-row-waits.md:57`
**Issue:** D9 and its acceptance criterion bound GitHub polls to at most once a minute.
Step 4 polls at once on the wake where `Continue` first appears, so two polls can fall within one minute.
Card 13 settles the focus question: a record carrying `Continue` on a dirty tree or an ungated head cannot poll on every wake, since the immediate poll fires only on the transition and later wakes keep the once-a-minute bound, and the planned row seeded with `Continue` before `Call` pins that.
The one-time extra poll is a departure from the answer key that neither Shared Decisions nor the test names.

## Rejected

- Seam markers and `intentionallyNil` entries (focus 1): card 11 lists all six tests marked `//lym:seam internal/landingshed#Deps` and both `intentionallyNil` maps, cards 13 and 14 list both maps for their own seams, and `Landing.PullRequest.Now` is not in envkit's `nilLegal`, so card 13's entry for it is needed; no card's seam test fails on its own seams.
- Record re-read and `UpdateRecord` use (focus 2): card 13 re-reads the record each wake before the `Continue`, `ClosedForContinue`, `GatedHead` and `MailedHead` judgments, and every row write after a record exists goes through `UpdateRecord`; `WriteRecord` is used only when none exists, which `pr-continue` refuses to write into, so no overwrite is possible.
- Closed-pull-request verdict (focus 3): a closed pull request ends Done on `Continue` only on a clean tree with the re-read head equal to `GatedHead`, and Finalize halts a closed record carrying `Continue` without `ClosedForContinue`; the two sides agree, bar the unreachable clause in the fourth finding.
- `ResolvePullRequest` host test (focus 5): the host rule is explicit for scheme, user-info and scp forms, compared case-insensitively, and a `ParseOwnerRepo` refusal of a github.com host is a halt before the missing-upstream and push-skip facts, so it never becomes a silent no; the test rows pin `https://GitHub.com/o/r`, `ssh://git@github.com:22/o/r.git` and the empty-user form.
- Doc, gate-key and recipe coverage (focus 6): the `pr-review` key is on card 4's target list with the gateslot doc, gatecli doc, `docs/dotnet-hub.md` and both PATTERN files, the shellguard and claudeengine docs are card 6 `Extends` targets, the pawl header and row sequence are on card 11, and the shedrecipe `like the PRReview flag` example is rewritten in card 11's Intent; the gaps are in the third finding.
