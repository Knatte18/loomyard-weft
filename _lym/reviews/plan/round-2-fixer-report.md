# Fixer report — plan, round 2

Every finding's premise holds against the code at HEAD; all six are fixed.
`lym loom validate-plan` reports `ok: true` after the changes, with informational findings only.

## Changes

- **F1 (BLOCKING), fixed differently.**
  The review put both `internal/pullrequest` and `internal/githubclient` on card 11.
  `githubclient` goes on card 8 instead: card 8's `ResolvePullRequest` calls `githubclient.ParseOwnerRepo`, so card 8 is the first card whose landingshed production code imports it, and its tier0 gate fails first without the entry.
  `TestToldGeometryInvariant_AllowlistOnly` runs `scankit.AssertImportAllowlistNoStale` (`internal/testkit/scankit/scankit.go:345`), which also fails on a listed import no production file uses, so each entry goes on the card that first imports it.
  - Card 8 gains an **Edit:** section with `internal/landingshed#landingshedAllowedImports` and `internal/landingshed#TestToldGeometryInvariant_AllowlistOnly`, and an Intent sentence adding `githubclient`.
  - Card 11 gains the same two targets and adds `internal/pullrequest`, noting that `githubclient` is already listed from card 8.
  - Card 14 gains the same two targets and adds `internal/parentdirective` and `internal/editdirective`.
- **F2 (BLOCKING), fixed as proposed.**
  Card 16's third guard clause also exempts `internal/testkit/envkit`, because its `LandingDeps` stub names `pullrequest.API`.
- **F3 (MEDIUM), fixed as proposed, with one wording change.**
  Card 11 lists `README.md` under **Edit:** and rewrites the README's PR-Review clause and the `internal/loomrecipe/doc.go` sentence `PR-Review always ends Done, so it carries no on_stuck`.
  The proposed README wording, that the operator discusses the pull request with an agent, is not made at card 11: the session arrives only in card 14, so card 11 says the row lands through a pull request when the resolver says so.
  Card 14 lists `README.md` and adds the agent session to that sentence.
- **F4 (MEDIUM), fixed as proposed.**
  - Card 11 step 6 removes a `ClosedForContinue` record and opens afresh only when `TaskHead()` differs from `GatedHead`.
    When the head equals it, the record is kept, steps 7 and 8 are skipped and the row returns Done.
  - `TestPRReview_Call` gets one row for each case, replacing the single remove-and-create row.
  - In card 13, `TestPRReview_Wait`'s row for the closed-and-`ClosedForContinue` verdict starts from a record that `Call` keeps and hands to `wait`.
  - To match, card 14's `ensureSession` never runs for a kept `ClosedForContinue` record, and `TestPRReview_Session` gains a row showing it starts no session.
- **F5 (MEDIUM), fixed as proposed, with the failure cases split.**
  - Card 13 creates `verifyGate.checkHead` and uses it in step 3 in place of `check`.
    `verifyGate.check` leaves card 13's **Uses:** list, and `verifyGate.readBase`, `verifyGate.markWait` and `gateslot#FailedTier` join it.
  - The mail names the failing tier from `FailedTier`, else the key's highest tier, or the build, together with the log path.
  - A timeout and a shell that could not start name no tier.
  - The failing-gate row asserts that the mail names the tier and the log path and contains neither `merging parent branch` nor `resume`.
- **F6 (LOW), fixed as proposed.**
  - The overview's Shared Decisions gain "One immediate poll on the continue flag", bounded to the first wake that reads `Continue` where the wake before it did not.
    That is one poll each time the flag appears, not one per run, because card 11 can remove a record after a reroute.
  - Card 13's step 4 cites that decision.
  - Card 13's row where `Continue` is written between wakes asserts that the `Status` count rises by exactly one, and that a dirty tree keeps later wakes to at most one poll a minute.

## Self-check

Changes made after the self-check:
- **Tier-fallback finding:** a timeout and a could-not-start each get their own reason naming no tier.
- **README clause:** card 11 now quotes the README's clause by its literal tail.
- **Closed-and-`ClosedForContinue` verdict:** it now ends Done only on a clean tree and halts on a dirty one, with a matching test row.
- **Card 13 sentence:** a new sentence states that `Call` hands a kept `ClosedForContinue` record to `wait`, skipping steps 7 and 8.
- **Stub wording in card 11:** card 11 now rewrites the landingshed package summary sentence, the `prreview_test.go` file comment and the `PRReview` type doc.

Each lens finding and its disposition:
- fix-claims, LOW: the README sentence card 11 quotes lacks the backticks around `PR-Review` — fixed as proposed.
- fix-claims, LOW, merged with fix-proposal's second finding: `checkHead` would name a guessed tier for a timeout or a shell that could not start — fixed as proposed.
- fix-consistency, LOW: no card adds the agent session to the README — fixed as proposed.
- fix-consistency, LOW: card 13 does not say that `Call` hands it the kept `ClosedForContinue` record — fixed as proposed.
- fix-proposal, LOW: the now-reachable closed-and-`ClosedForContinue` verdict ends Done on a dirty tree — fixed as proposed.
- fix-proposal, LOW: the tier fallback also applies when the shell never started — merged with the fix-claims finding above.
- fix-affected-set, LOW: no card rewrites `internal/landingshed/doc.go:1`'s `the PR-Review stub` — fixed as proposed.
- fix-affected-set, LOW: no card rewrites `prreview_test.go`'s file comment or `PRReview`'s type doc — fixed as proposed.

Every lens returned a report.

## Disputed

None.

## Deferred

None.
