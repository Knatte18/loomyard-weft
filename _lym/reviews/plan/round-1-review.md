---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/11-pr-review-row-opens.md (also cards 13 and 14)
    summary: The nested Deps.PullRequest seams leave the wire seam tests failing; their intentionallyNil maps and seam markers are edited by no card, so card 11 cannot pass its own tier0 gate.
    fix: "In card 11, move internal/battencli#TestWire_EverySeamFilled, internal/loomcli#TestWire_DarnRecipe and internal/loomcli#TestWire_EverySeamFilled from Uses to Edit, and drop the same three lines from card 4's Uses, since they would otherwise be a uses-later-target. Add Edit targets internal/loomcli#TestLandingDeps_EveryFieldPopulated, internal/battencli#intentionallyNil and internal/loomcli#intentionallyNil. In the Intent, give every test that carries //lym:seam internal/landingshed#Deps the marker //lym:seam internal/landingshed#PullRequestDeps; this means battencli's TestWire_EverySeamFilled, loomcli's TestWire_EverySeamFilled, TestWire_DarnRecipe and TestWire_BeamRecipe, and TestLandingDeps_EveryFieldPopulated, whose marker moves here from card 14. Also give every Landing.PullRequest.* seam that wire() leaves nil an entry in both intentionallyNil maps, with the reason the sibling Landing.* entries give. Add internal/battencli#intentionallyNil and internal/loomcli#intentionallyNil to the Edit lists of cards 13 and 14, and say in each Intent that the seams it adds get the same entries. A combined validate-plan --overlay run of these target changes reports no blocking finding."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-pr-review-row-waits.md (PRReview.wait, step 4 and the open-with-Continue verdict)
    summary: wait takes the record by value at its start and never re-reads it, so a Continue the agent writes mid-wait through lym loom pr-continue is never seen, and a whole-record write of the stale copy can erase it.
    fix: "In card 13's Intent, say that each wake re-reads the record with pullrequest.ReadRecord before step 3, and judges the Continue and ClosedForContinue verdicts and the GatedHead and MailedHead reads on that fresh copy. Say that every record write the row makes goes through pullrequest.UpdateRecord, mutating only the fields it owns. That covers card 11's step-8 write and card 14's session-field write, so a concurrent pr-continue is never overwritten. Add pullrequest#ReadRecord to card 13's Uses. Add a TestPRReview_Wait row whose fake writes Continue into the record between two wakes and asserts that the next wake closes the pull request and is Done."
  - id: F3
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md (Shared Decisions, A covering pause holds the row in process) and _lym/plan/13-pr-review-row-waits.md (holdWhileCovered)
    summary: D9 says a covering pause stops the session and ends the call with a transient mark; the plan replaces that with an in-process hold, with no Added after Discussion entry recording the change.
    fix: "Ask the parent (lym creel post --parent --expect-reply) to settle D9's pause step. The plan argues that the transient mark does not fit PATTERN-transient-stop, whose classes are failures, and that a held entry point waits in process as pauserecord.WaitUncovered and shuttle's waitWhileCovered do. Once the parent records the in-process hold in the decision record as an Added after Discussion entry, keep cards 13 and 14 as written. If the parent keeps D9, rewrite card 13's pause step to return a pause error that internal/shedtransient classifies into a new shedengine transient class, and add that class and mapping as targets."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/13-pr-review-row-waits.md (closed-and-not-merged verdict) and _lym/plan/15-finalize-through-pull-request.md (pullRequestLanding, Continue with a closed pull request)
    summary: A closed pull request with Continue but no ClosedForContinue ends Done, and Finalize then lands it locally, without the clean-tree and gated-head checks the open-with-Continue verdict demands.
    fix: "In card 13, make the closed verdict Done only when the record carries ClosedForContinue, or carries Continue on a clean tree whose re-read head equals GatedHead, recording ClosedForContinue in that case. Otherwise keep waiting, letting step 3 gate a moved head first, or halt as the closed halt. In card 15, take the operator's-ending branch only for a record carrying ClosedForContinue, and treat Continue alone with a closed pull request as the closed-without-a-merge halt. Add a TestPRReview_Wait row for a closed pull request with Continue on a dirty tree, and one with an ungated head."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-pull-request-mode-resolver.md (ResolvePullRequest origin checks)
    summary: The resolver never says how it tells a non-github.com host from a github.com origin ParseOwnerRepo refuses, so refused github.com shapes can become a silent no under parent on, which D4 forbids.
    fix: "In card 8's Intent, name the host test. The host is the part after any scheme:// and any user-info@, up to the first : or /, or the host of a user@host: scp form. An origin is on github.com when that host equals github.com. Only then does a ParseOwnerRepo refusal give the halt; any other host gives the not-github.com fact. Add TestResolvePullRequest rows under parent on for https://@github.com/o/r, ssh://git@github.com:22/o/r.git and git://github.com/o/r: each halts. Add a row for ssh://git@gitlab.com/o/r.git, which is a no naming the origin."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/13-pr-review-row-waits.md (wake step 4)
    summary: Step 4 reads Status at once on every wake while the record carries Continue, so a dirty tree or ungated head after the flag polls GitHub as often as once a second, breaking D9's once-a-minute poll bound and the card's own test row.
    fix: "In card 13's step 4, read Status at once only on the first wake that sees Continue newly set, and at most once a minute after that, or skip the poll until the tree is clean and the head equals GatedHead. Make the once-a-minute TestPRReview_Wait row also cover a record carrying Continue on a dirty tree across several wakes."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/05-shuttle-unawaited-session.md
    summary: StartSession creates a run with an empty OutputFiles, which falsifies wait.go's stated precondition that no such run exists; the card lists neither wait.go nor, except under Uses, the package doc it edits.
    fix: "In card 5, add internal/shuttleengine/wait.go to Edit. Move internal/shuttleengine/doc.go from Uses to a new Extends group. In the Intent, rewrite the precondition in wait.go's file header and its restatements near classifyDeadlineExpiry and the Wait loop: an empty OutputFiles exists only on an unawaited run, which never reaches Wait or Attach. Also state what reed's DoneWhen receives for an unawaited spec. A combined validate-plan --overlay run of these target changes reports no blocking finding."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/10-pr-review-stencil.md
    summary: The card edits contracts/stencils#registry, the stencilstore.Registry type, but the list it adds the stencil to is the entries variable.
    fix: In card 10's Edit list, replace `contracts/stencils#registry` with `contracts/stencils#entries`, and in the Intent say it lists the stencil in entries. An overlay run of this change reports no blocking finding.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/04-pr-review-config-keys.md, _lym/plan/06-publish-shell-guard.md, _lym/plan/11-pr-review-row-opens.md
    summary: Several cards leave doc text they falsify off their target lists or out of their Intent.
    fix: "Card 4: add Extends internal/gatecli/doc.go, pattern/PATTERN-test-speed.md and pattern/PATTERN-gate-slot.md. In the Intent, add pr-review to gatecli's example gates block and to the key table of PATTERN-test-speed.md (PR-Review's gate over the agent's commits, diff). Add PR-Review to the site list of PATTERN-gate-slot.md. Card 6: move internal/shellguard/doc.go and internal/shuttleengine/claudeengine/doc.go from Uses to a new Extends group. Card 11: add Edit internal/loomrecipe/doc.go, whose pawl sequence gains PR-Review between Beam-Describe and Finalize. Its Intent also rewrites the row list in pawl-recipe.yaml's header to include PR-Review. It drops the PRReview flag example from the sentence on auto-accept values in internal/shedrecipe/doc.go. A combined validate-plan --overlay run of these target changes, with F1, F7 and F8, reports no blocking finding."
  - id: F10
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/11-pr-review-row-opens.md (shedrecipe paragraph)
    summary: D3 says the PRReview registry entry takes no config key, but card 11 lets it take an optional gate_key.
    fix: "In card 11, make prReviewEntry refuse every config key: configRejectUnknown(cfg) with no allowed key, with prReviewKeyWayForward reading way forward: remove the key; a PRReview row takes no config. It still builds its deps through landingRowDeps, whose default gate key is the lower-cased row name, pr-review. Change TestPRReviewEntry so that gate_key and the retired pr_review_setting are each refused. Change the shedrecipe doc sentence to say the row takes no config key."
  - id: F11
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/16-teardown-closes-pull-request.md (TestPullRequestGuard_OnlyLandingAndTeardown)
    summary: The guard scans only the PullRequests mutating calls, but a pull request is also an issue and can be closed through Issues.Edit with nothing to flag it.
    fix: "In card 16's guard test, also flag a non-test file outside internal/pullrequest and internal/hubissues that calls .Issues.Edit(. Or have the Intent state the bound: the scan is a tripwire over the PullRequests calls only, and an issue-endpoint close is left to review."
  - id: F12
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-publish-shell-guard.md (publish row comment, the bound)
    summary: The row's stated bound lists only shell forms that slip the deny, and leaves out lym verbs that push the task branch from the session, such as lym fabric push and the detached push a records commit fires.
    fix: "In card 6's Intent, extend the row comment's bound. A lym verb that pushes the task branch, such as lym fabric push or the detached both-sides push a run-record commit fires, also reaches origin from the session. The row gates what lands, not what reaches the pull request."
---

### [BLOCKING] Nested PullRequest seams break the wire seam tests

**Location:** `_lym/plan/11-pr-review-row-opens.md`, cards 13 and 14.

**Issue:** `wire()` leaves `Env.Landing`'s func seams nil and lists each one in `intentionallyNil` (`internal/battencli/wire_seams_test.go:73-80`, `internal/loomcli/wiring_seams_test.go:16-21`).
Each test fails on a nil path missing from that map, and the tier0 seam scan refuses a test whose markers miss a struct in the `Env` set.
Cards 11, 13 and 14 add `Landing.PullRequest.*` func seams, but they list these tests only under Uses, and no card's Intent touches the maps or adds the `#PullRequestDeps` marker before card 14.

### [BLOCKING] The continue flag is never seen during a live wait

**Location:** `_lym/plan/13-pr-review-row-waits.md`, `PRReview.wait`.

**Issue:** `wait(ctx, api, rec pullrequest.Record)` takes the record once, and the card never re-reads it.
Card 14 starts the session after the record is written, so the agent's `lym loom pr-continue` always lands mid-wait.
The in-memory copy never carries `Continue`, so the operator's ending, the main way a review ends under the added-after-Discussion decision, cannot fire.
The test row seeds `Continue` before `Call`, so it passes regardless.
Card 11 writes the record with `WriteRecord` after a read, and card 14 does not say how it writes, so either can overwrite a `Continue` written between its read and write.

### [BLOCKING] D9's pause step is replaced without a recorded decision

**Location:** `_lym/plan/00-overview.md` Shared Decisions, and card 13's `holdWhileCovered`.

**Issue:** D9 says a covering pause stops the session and "ends the call with a transient mark (`PATTERN-transient-stop`)".
The overview holds the row in process instead.
Its argument has substance: shedengine's transient classes are infrastructure failures (`internal/shedengine/transient.go`), and held entry points do wait in process (`pauserecord.WaitUncovered`, shuttle's `waitWhileCovered`).
But the PATTERN does not forbid the decision outright, since `internal/shedtransient` could map a pause error to a new class.
The decision record carries no Added after Discussion entry for the change, and under the rubric an uncarried Decision step stays a finding.
The parent settles it.

### [MEDIUM] A closed pull request with Continue skips the continue checks

**Location:** card 13's closed verdict, and card 15's operator's-ending branch.

**Issue:** The open-with-`Continue` verdict closes the pull request only on a clean tree whose head equals `GatedHead`.
The closed verdict accepts `Continue` alone, so a close by a collaborator, or by an agent past the deny, ends Done on a dirty tree or an ungated head.
Card 15 then lands that record locally.

### [MEDIUM] The resolver's github.com host test is undefined

**Location:** `_lym/plan/08-pull-request-mode-resolver.md`.

**Issue:** `ownerRepoPath` returns the same "not a github.com remote URL" error for a gitlab URL and for a refused github.com shape (`internal/githubclient/parseownerrepo.go:46-47`).
The resolver must tell them apart: D4 makes the first a no and the second a halt.
The card names no host test, and pins only `https://github.com/onlyowner`.

### [MEDIUM] The Continue poll breaks the once-a-minute bound

**Location:** card 13, wake step 4.

**Issue:** "At most once a minute, and at once when the record carries `Continue`, it reads `Status`."
While `Continue` stands on a dirty tree or an ungated head, every wake polls, and wakes start a second apart.
D9 and its acceptance criterion require polling GitHub at most once a minute, as does the card's own test row.

### [MEDIUM] Card 5 falsifies shuttle's empty-OutputFiles precondition

**Location:** `_lym/plan/05-shuttle-unawaited-session.md`.

**Issue:** `internal/shuttleengine/wait.go`'s header states that no `*Run` with an empty `OutputFiles` ever exists, because `Spec.validate` refuses one, and two later comments restate it.
That precondition underlies the completion-signal rule `PATTERN-shuttle-provider-seam` cites.
`StartSession` creates exactly such a run, yet the card lists neither `wait.go` nor, except under Uses, the package doc it says it extends.

### [MEDIUM] Card 10 names the wrong stencils symbol

**Location:** `_lym/plan/10-pr-review-stencil.md`.

**Issue:** The name-to-default list is `entries` (`contracts/stencils/stencils.go:450-456`).
`registry` (`stencils.go:597`) is the type implementing `stencilstore.Registry`, which does not change.

### [MEDIUM] Doc text the cards falsify is left unlisted

**Location:** cards 4, 6 and 11.

**Issue:** The `pr-review` gate key and its site are missing from three places that list every key and site: `pattern/PATTERN-test-speed.md`'s key table, `pattern/PATTERN-gate-slot.md`'s site list and `internal/gatecli/doc.go`'s gates example.
Card 6 edits the shellguard and claudeengine docs but lists them only under Uses.
Card 11's pawl change falsifies the row sequence in `internal/loomrecipe/doc.go:30` and the row list in `pawl-recipe.yaml`'s header.
It also leaves `internal/shedrecipe/doc.go:94`'s "like the PRReview flag" example naming a removed flag.

### [MEDIUM] Card 11 lets PRReview take a gate_key that D3 rules out

**Location:** `_lym/plan/11-pr-review-row-opens.md`, shedrecipe paragraph.

**Issue:** D3 says the `PRReview` registry entry "drops `pr_review_setting` and takes no config key".
Card 11 has it take the optional `gate_key`, which is scope the decision record does not license.
The default gate key from `landingRowDeps` is already `pr-review`.

### [LOW] The pull-request guard misses the issue-endpoint close

**Location:** card 16, `TestPullRequestGuard_OnlyLandingAndTeardown`.

**Issue:** The scan flags `.PullRequests.Create(`, `.Edit(` and `.Merge(` only, so `.Issues.Edit(` with state closed escapes it, and the card does not state that bound.

### [LOW] The publish deny's stated bound under-reports

**Location:** card 6, the publish row comment.

**Issue:** The bound names `bash -c`, scripts, `env`, aliases and `hub`, but not the lym verbs that push the task branch from the session, which D8's bound itself names.

## Rejected

- affected-set F4 (card 12 retired-key comments stale between cards 11 and 12): the lens itself notes card 12 covers all four, so nothing is left uncovered.
- affected-set F1's attach, waitshadow and run.go items as a code defect: the guard-reach lens traced that the wait and attach sites see only validated non-empty specs and `sessionread.go:124` guards with `len>0`, so only the falsified precondition text stands (F7).
