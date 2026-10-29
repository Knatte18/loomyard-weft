# Fixer report — plan, round 1

F3 is a `decision` finding; the parent settled it, so it is left out here.
`lym webster validate` over the changed plan reports one blocking finding, `plan-unapproved`, which is expected before approval, and no target finding.

## Changes

- **F1** (fixed as proposed): card 11 moves `battencli#TestWire_EverySeamFilled`, `loomcli#TestWire_DarnRecipe` and `loomcli#TestWire_EverySeamFilled` from Uses to Edit, and adds Edit targets `loomcli#TestLandingDeps_EveryFieldPopulated`, `battencli#intentionallyNil` and `loomcli#intentionallyNil`.
  Card 4 drops those three Uses lines.
  Card 11's Intent gives every test that carries `//lym:seam internal/landingshed#Deps` the `#PullRequestDeps` marker.
  It also gives both `intentionallyNil` maps an entry for each `Landing.PullRequest.*` func seam it adds, with the sibling reason.
  Card 14 drops its marker sentence, since the marker moved to card 11.
  Cards 13 and 14 add both maps to Edit, with the same entry rule for the seams each adds.
- **F2** (fixed as proposed): in card 13, each wake re-reads the record with `pullrequest.ReadRecord` before step 3 and judges `Continue`, `ClosedForContinue`, `GatedHead` and `MailedHead` on that fresh copy.
  Every record write goes through `pullrequest.UpdateRecord` and mutates only the fields it owns.
  `pullrequest#ReadRecord` is added to card 13's Uses.
  A new `TestPRReview_Wait` row writes `Continue` into the record between two wakes.
  Two edits outside card 13:
  - Card 11's step 8 now updates an existing record through `UpdateRecord`. It keeps `WriteRecord` only for a first record, which pr-continue cannot write into because `UpdateRecord` returns `ErrNoRecord` on an absent record.
  - Card 14's session-field write goes through `UpdateRecord`.
- **F4** (fixed as proposed): in card 13, the closed verdict is Done only on `ClosedForContinue`, or on `Continue` with a clean tree and a gated head, which records `ClosedForContinue`.
  `Continue` on a dirty tree or an ungated head keeps waiting; otherwise the result is the closed halt.
  New test rows cover a dirty tree and an ungated head.
  In card 15, only `ClosedForContinue` is the operator's ending, and `Continue` alone on a closed pull request is the closed-without-a-merge halt. Card 15 gains a test row for that halt.
- **F5** (fixed as proposed): card 8 names the host test.
  It also adds halt rows for `https://@github.com/o/r`, `ssh://git@github.com:22/o/r.git` and `git://github.com/o/r`, and a no row for `ssh://git@gitlab.com/o/r.git`.
- **F6** (fixed as proposed): in card 13, step 4 polls at once only when `Continue` is newly set, and at most once a minute otherwise.
  The once-a-minute row now covers `Continue` on a dirty tree over several wakes.
- **F7** (fixed as proposed, one site added): card 5 adds `internal/shuttleengine/wait.go` to Edit and moves `internal/shuttleengine/doc.go` into a new Extends group.
  Its Intent rewrites the precondition in four places: the `wait.go` file header, the comments above `classifyDeadlineExpiry` and `finishedDespiteMechanismFailure`, and the same restatement inside `attach.go`'s `dispositionCandidate`.
  `dispositionCandidate` is the added site; it is also added as an Edit target.
  The Intent also states what reed's `DoneWhen` receives for an unawaited spec.
- **F8** (fixed as proposed): card 10 targets and names `contracts/stencils#entries`.
- **F9** (fixed as proposed):
  - Card 4 extends `internal/gatecli/doc.go`, `pattern/PATTERN-test-speed.md` and `pattern/PATTERN-gate-slot.md`, with the `pr-review` additions in its Intent.
  - Card 6 moves the shellguard and claudeengine docs to Extends.
  - Card 11 adds `internal/loomrecipe/doc.go` to Edit, rewrites its pawl sequence and `pawl-recipe.yaml`'s header row list, and drops the `PRReview` flag example from the shedrecipe doc.
- **F10** (fixed as proposed): in card 11, `prReviewEntry` refuses every config key with the way forward `remove the key; a PRReview row takes no config`.
  `TestPRReviewEntry` refuses `gate_key` and `pr_review_setting`, and the shedrecipe doc says the row takes no config key.
- **F11** (fixed as proposed, the guard option): card 16's guard also flags `.Issues.Edit(` outside `internal/pullrequest` and `internal/hubissues`.
  `internal/hubissues/hubissues.go` is the only current non-test caller.
- **F12** (fixed as proposed): card 6's row comment adds that lym verbs such as `lym fabric push` and the detached records push reach origin from the session, so the row gates what lands, not what reaches the pull request.

## Self-check

- fix-consistency, LOW, card 13's Bound named "closed" as always ending the wait: fixed. The Bound now says a closed pull request carrying `Continue` waits until the tree is clean and its head is gated.
- fix-affected-set, MEDIUM, the pawl row list in `internal/loomcli/doc.go` was left stale: fixed. Card 11 moves `internal/loomcli/doc.go` from Uses to Edit and names PR-Review in that sentence.
- fix-affected-set, MEDIUM, the `gates:` block in `docs/dotnet-hub.md` lacked `pr-review`: fixed. Card 4 extends that file and adds the key.
- fix-affected-set, LOW, `internal/loomshed/doc.go:46` lists no `pr-review`: disputed, premise false. That sentence lists the commands loomshed's `NewVerifyGate` is told for loomshed's fix rows and writers. PR-Review's gate is landingshed's `verifyGate` (card 13, `newVerifyGate(deps)`), which that sentence never covers.
- fix-affected-set, LOW, `internal/shuttleengine/doc.go:186`'s stop sentence and `Spec.validate`'s doc were left stale: fixed. Card 5's Intent rewrites both.
- fix-claims, LOW, card 5's "relaunches it" ignored reed's pause hold: fixed. Card 5 now says reed "treats it like any other non-live strand".
- fix-proposal, MEDIUM, a closed pull request carrying `Continue` on a dirty tree stops the agent, and nothing restarts it: fixed. Card 14 resumes the session through `ensureSession` after a continue read that keeps waiting, open or closed, and gains test rows for both.
- fix-proposal, LOW, card 8's host comparison was case-sensitive: fixed. Card 8 compares case-insensitively and adds a halt row for `https://GitHub.com/o/r`.
- fix-proposal, LOW, "newly set" was undefined, and the poll row was ambiguous: fixed. Card 13 defines "newly set" against the previous wake's copy, or the starting copy, and the row seeds `Continue` before `Call`.

## Disputed

- fix-affected-set LOW (`internal/loomshed/doc.go:46`): premise false, evidence above. This is a self-check finding; no review finding is disputed.

## Deferred

Nothing deferred.
