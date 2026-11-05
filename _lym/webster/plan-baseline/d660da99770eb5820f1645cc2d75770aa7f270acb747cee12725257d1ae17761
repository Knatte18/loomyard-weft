# Card 2 — landing-record

**Create:**
- `internal/pawlengine#LandingRecord`
- `internal/pawlengine#LandingPullRequest`
- `internal/pawlengine#ReadLanding`
- `internal/pawlengine#UpdateLanding`
- `internal/pawlengine#DropLandingPullRequest`
- `internal/loomengine#BeamLandingPath`
- `internal/loomengine#BeamLandingLockPath`
- `internal/pawlengine_test#TestLandingRecord`

**Edit:**
- `internal/loomengine#TestLandingAndApprovalAccessors`

**Extends:**
- `internal/pawlengine/doc.go`

**Uses:**
- `internal/loomengine#LandingDir`
- `internal/pawlengine#ReadyFor`

**Intent:** Add the tracked beam landing record of D4, beside the ready record in the beam worktree's landing directory.

- `LandingRecord` holds `Slug` (the beam it was written for), `DescribedAt time.Time` (when `lym beam describe` last wrote the description for this beam, zero for never) and `PullRequest *LandingPullRequest`, nil while none is recorded. `LandingPullRequest` holds `Number int`, `URL string`, `PushedHead string` (the beam head it was last pushed with) and `AnnounceHead string` (the beam head D7's member-landing notice announces from). JSON keys are snake_case; the record goes through `internal/state`, so an unknown key survives (`PATTERN-tolerant-records`).
- `ReadLanding` reads the record at path under the lock; an absent file and a record naming another slug read as not found (a pair forked from the beam inherits the beam's records, as `ReadyFor` filters).
- `UpdateLanding` applies mutate to the record for slug under one lock and writes it, creating the lock's directory: a missing record and one naming another slug start from `LandingRecord{Slug: slug}`, so an inherited record is replaced by the beam's own and never removed. A mutate error writes nothing.
- `DropLandingPullRequest` sets `PullRequest` to nil on a record naming slug and reports whether one was dropped; a record naming another slug, and an absent file, are left untouched and report false.
- `loomengine.BeamLandingPath` is `beam-landing.json` in `LandingDir`, under a new segment constant declared once beside `beamReadyFileName`; `BeamLandingLockPath` is `lock.TrackedFileLockPath(BeamLandingPath(l), ".lock")`, under `.lym/locks/` (`PATTERN-tracked-file-locks`, `PATTERN-durable-vs-ephemeral-state`). The verbs that write the record commit the landing directory themselves; this card adds no commit.

Tests (tier 0): the new `TestLandingRecord` covers a round trip of a record with and without a pull request, an update over an absent file, an update over a record naming another slug (replaced, slug set), a read of a record naming another slug (not found, file kept), a drop on the beam's record (pull request gone, `DescribedAt` kept, true), a drop on another slug's record and on an absent file (false, file unchanged), and a mutate error (nothing written). `TestLandingAndApprovalAccessors` gains rows for the two paths: the record under `_lym/landing`, its lock under the checkout's `.lym/locks/` and never beside the tracked file.

**ImpactSummary:** two new loomengine accessors and a new pawlengine record type with its functions; no existing caller changes.

**Commit:** 2: Add the beam landing record beside the ready record
