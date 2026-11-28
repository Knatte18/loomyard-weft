---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed text does not fit the section's format: summaryparser.ValidateDescription takes only one-way or two-way as the door and one word as the blast radius, so the section holds Door one-way and Blast radius config with the conditions in prose below. The proposed loom wiring is not named as a reader of dobbyengine.LoadConfig, since no call in internal/loomcli reaches it."
  - id: F3
    disposition: fixed-differently
    reason: "The later reads post through deps.RunWatch.Post, not deps.RunNotices.Post: a run notice carries the run's topic tag, so a second one supersedes the unread one the test already posted. The failed post the review found no way to reach in a hub is pinned by a direct tellRefusal test."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

## Changes

- F1: `autonomyName()` in `internal/beamcli/keynames.go` renders `beam.autonomy` in the refusal mail and in the post-failure warning.
  The test's `refusalPrefix` follows.
- F2: rewrote `## Merge danger` in `_lym/landing/summary.md`, differently as above.
  Read for the fix: the non-test callers of `dobbyengine.LoadConfig` are the dobby, its view reader, adopt's fix loop and `lym config beam`'s load check.
  `lym loom validate-description` passes.
- F3: the refused row of `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` now fixes the value, breaks it again and expects the refusal mail a second time, differently as above.
- F4: added the `unattended` row to `TestRunAutonomy_ForcesTheRunsKeysOverARealHub`.
  The review left the pawl seam and the Slack wiring unverified; both can be driven over a real hub, so `TestPawlDeps_AutonomousReadsUnattendedAsOn` and `TestWireProduction_OrchAutonomousReadsUnattendedAsOn` pin them.
- F5: `hubgeom.AutonomyWithRefusal` also returns `read`, and `tellRefusal` ends an episode only on a successful read.
  Docs, tests and the summary follow.
- F6: `tellRefusal` records the episode under the lock and posts unlocked; a failed post drops only its own episode.
  A mail of the same run stamped during the post may arrive ahead of the refusal mail, which the doc and the summary now state.
- Gate repair (F5, F6): the `tellRefusal` doc in `internal/beamcli/dobbydeps.go` and the `AutonomyWithRefusal` lines in `internal/hubgeom/doc.go` now hold one sentence per line, each under the line limit.
  Only comments changed.

Gate: `lym gate test --tier 0 --tier 1` over `internal/beamcli`, `internal/hubgeom`, `internal/loomcli` and `internal/slackcli`, each re-run after its last change, with no failures.

## Self-check

- fix-code-correctness, LOW, refusal mail no longer guaranteed ahead of the run's other mails: fixed differently, by stating the ordering in the `tellRefusal` doc and the summary; restoring the order would hold mails behind the post, which F6 removes.
- fix-code-correctness, note, no test pins F5 in `tellRefusal`: fixed, `TestRunAutonomyStamps_PostsARefusalOncePerEpisode`.
- fix-consistency, BLOCKING, Merge danger fails the description validator: fixed (merged with fix-proposal 1).
- fix-consistency, LOW, the warning still names the key informally: fixed (merged with fix-affected-set).
- fix-consistency, LOW, ordering claim of the unlocked post: fixed differently (merged with the first line).
- fix-proposal, BLOCKING, Merge danger format: fixed.
- fix-proposal, LOW, pawl seam and Slack wiring unpinned: fixed, one test each.
- fix-proposal, LOW, ordering claim of the unlocked post: fixed differently (merged with the first line).
- fix-proposal, LOW, F5 behaviour unpinned: fixed (same test as above).
- fix-affected-set, LOW, the warning still names the key informally: fixed.

## Disputed

## Deferred

## Rulings
