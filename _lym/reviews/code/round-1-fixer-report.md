# Fixer-report — code review round 1

Every finding F1–F6 is fixed as proposed.
The gate (`lym gate test`, tiers 0 and 1, over `hubregistry`, `configreg`, `creelconfig`, `creelengine` and `creelcli`) passes.
Commits: F1, F2, then F3–F6 together (their edits overlap in `cli_test.go`, `repository.go` and `configreg.go`), then a follow-up for the self-check.

## Gate repair

The gate flagged wrapped and over-long comment lines in `configreg.go`, `configreg_test.go`, `hubregistry/repository.go` and `hubregistry/repository_test.go`.
Each is now one sentence per line, with no line at 120 characters.
A second gate run flagged one line left at 120 characters in `markUnresolvable`'s doc comment; it is now two sentences.
No code changed in this repair.

## Changes

- **F1** — `internal/creelengine/crosshub.go`: `widestStampedLine` sets `SendingRepository` to the longest legal GitHub name (39-character owner, slash, 100-character repository).
  `verbs_test.go`: the `stampedOverhead` of the over-budget row now includes that repository, so the existing +1 row fails without it.
  A new check posts a line exactly at the budget and expects it accepted.
  `creelengine/doc.go` states that the longest repository name is counted.
- **F2** — `internal/creelengine/journal_test.go`: the inbound record of the replay row and of the retention-carry row carries `SendingRepository`.
  The assertions cover the folded message after replay, after the carry, and after the crash between carry and delete, and a record with none stays empty.
- **F3** — `internal/creelcli/cli_test.go`: the cross-hub read test has a second inbound message with no repository.
  It asserts `sending_repository` present on the first message and absent on the second.
- **F4** — `internal/configreg/configreg_test.go`: a test pins that the `creel` module registers `TrustedHubsRepositoryReinterpretation` with a rewrite.
  A second test pins `markUnresolvable`.
  New `internal/hubregistry/repository_test.go` (tier 1, a git prime per hub) covers `LookupRepository`: a recorded shortname, an unrecorded one, an ambiguous one, a hub with no origin and a corrupt record.
  `hubregistry/testmain_test.go` now sets the hermetic git environment for it.
- **F5** — `internal/creelconfig/reinterpret.go`: new `ErrShortnameUnresolvable`.
  The rewrite drops an entry only when the resolver wraps it, and returns any other resolve error, so `hubreconcile` leaves the reinterpretation unapplied and retries.
  `internal/hubregistry/repository.go`: new `ErrNoRepository`, matched by every `RepositoryName` refusal.
  `internal/configreg/configreg.go`: `repositoryOfShortname` marks `ErrUnresolved` and `ErrNoRepository` as unresolvable and passes the `UserStateDir` and unreadable-record errors through.
  `reinterpret_test.go`: the scripted resolver wraps the sentinel, and a new subtest covers a resolver failing with an unrelated error.
  `creelconfig/doc.go` states the new rule.
- **F6** — `internal/hubregistry/repository.go`: `RepositoryName` refuses an owner or repository outside letters, digits, hyphen, underscore and dot, over 39 or 100 characters, or equal to `.` or `..`, with a reason that names the remote.
  `repository_test.go` covers the limits, a name imitating a stamp and a repository of only dots.
  `cli_test.go`: the `creel hubs` row gets a hub with a forged origin and shows `no_repository` and untrusted, even when `trusted_hubs` lists the forged name.

## Self-check

Forks: fix-code-correctness, fix-consistency, fix-proposal, fix-affected-set.
All four returned a report.

- fix-code-correctness, MEDIUM: a non-unresolved resolve error fails the whole hub-wide reconcile (`walk.go:249`) — disputed, every fix worse (see Disputed).
- fix-code-correctness, LOW: the at-the-limit fit check also passes with the F1 defect in place, and the +1 row is the regression guard — no change: the finding states no defect, and the +1 row already pins F1.
- fix-proposal, MEDIUM: the same reconcile-wide failure as the correctness finding — merged with it, disputed.
- fix-proposal, LOW: every `HubOriginURL` error is classed as `ErrNoRepository`, so a transient git-config read failure still drops the entry — deferred (see Deferred).
- fix-proposal, LOW: `.` and `..` pass `validName` — fixed with my own change: it is the review's F6 rule plus the two reserved names, with a test row.
- fix-consistency, LOW: `creelconfig/doc.go` still said it drops a shortname that resolves to none — fixed.
- fix-consistency, LOW: `creelengine/doc.go` did not say the repository is counted in the stamped line — fixed.
- fix-affected-set: no findings.

## Disputed

- **Fork finding, whole-walk failure on a non-unresolved resolve error** (fix-code-correctness and fix-proposal).
  Ground: every fix worse.
  The reviewer's F5 fix asks for exactly this: a transient failure must not mark the reinterpretation applied.
  `hubreconcile/reinterpret.go` writes the marker whenever the rewrite returns no error, so every alternative that skips the entry and continues either marks the reinterpretation applied, which loses the retry F5 asks for, or needs a new unsettled-entry channel in `hubreconcile`.
  The error carries a way forward: `refuseUnreadable` names the record file to remove, and `UserStateDir` names `XDG_STATE_HOME` and `HOME`.
  The cost is a reconcile that fails until the machine state is readable again, which the operator can see.

## Deferred

- **Fork finding, `RepositoryName` classes a failed read of the prime's git config as `ErrNoRepository`** (fix-proposal, LOW).
  Telling a missing `origin` remote from a read failure needs a sentinel for it exported from `gitrepo`, whose `RemoteURL` wraps go-git errors with no such marker.
  That adds API to a package the review does not touch, for a local-file read failure that is rare, and `creel hubs` and `Deliver` already treat every refusal as untrusted.
  An operator decision on whether `gitrepo` gains that sentinel settles it.
