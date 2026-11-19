---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: ruling
    reason: "The parent recorded the rows-only narrowing (decision notice #2123, \"Row subscriptions only, no state subscriptions\"). The code already holds rows only, so the ruling asks for no change and carries no flag."
  - id: F8
    disposition: fixed
---

## Changes

Each fix is its own commit, except F2, F4 and F8, which share one test file and one commit.

- **F1** (`internal/gatecli/cardfixer.go`, `internal/gatecli/doc.go`, `_lym/landing/summary.md`): removed the `SetNotifier` call in `newStrandRig`, the `gateProgram` constant, the `orchcli` import and the two added comment lines, the two doc lines in `gatecli/doc.go`, and the summary's "Gate card fixer" bullet, its fallback sentence and its "To check by hand" item.
- **F2** (`internal/loomcli/rownotice_test.go`): added `TestNewShed_WrapsTheRowsNotifyParentOnNames`, a table over a wired `loomCLI` with rows `EmptyList`, `RecipeRow` and `UnknownRow`.
  It pins that an empty list wraps nothing, a recipe row's producer is a `rowNoticeProducer` over `shedrun.RowNoticeRecord` and the run's status paths, and an unknown name returns a nil shed and the refusal.
- **F3** (`orch-role.md`, `loomcli/doc.go`, `rowNoticeProducer` and `verdictCount` comments, `shedrun.RowNotice`, `config.go`, `template.yaml`, `docs/overview.md`, `_lym/landing/summary.md`): stated that a re-call after an escalation's answer or a resumed halt counts as a new visit and mails again; a call after a yield or a pause mails nothing.
  I added `config.go` and `docs/overview.md`, which carry the same once-per-reach wording as the files the review named.
- **F4** (`internal/loomcli/rownotice_test.go`): replaced `TestWatchRows_FailedNoticeLeavesTheCall` with `TestWatchRows_NoticeNeverStopsTheCall`, a table with rows `RefusedPost` and `MissingStatus`.
  `countingProducer` now returns a pointer and an error.
  The test asserts pass-through of outcome, pointer and error, that the notice went out with zero inner calls made, that a second call posts no second notice, and that a missing status posts nothing and still runs the inner call.
- **F5** (`internal/shedrun/paths_test.go`): added the `RowNoticeRecord` row beside `WakeRecord`.
- **F6** (`internal/loomengine/config_test.go`): added the `notify_parent_on overridden` row to `TestLoadConfig_Loads`.
- **F8** (`internal/loomcli/rownotice_test.go`): `TestWatchRows_TellsParentOncePerVisit` now checks every notice's prefix and fails fast when there is none.

A later gate round found four comment lines wrapped at a fixed column or 120 characters or longer: `internal/loomcli/doc.go`, `internal/loomcli/rownotice_test.go`, `internal/loomengine/config.go` and `internal/shedrun/rownotice.go`.
I rebroke each at sentence boundaries, one sentence per line, in a follow-up commit.

`lym gate test` over `internal/loomcli`, `internal/shedrun`, `internal/loomengine` and `internal/gatecli` passed with 0 failed.
`_lym/landing/summary.md` is edited in the working tree and not committed, since no git command runs against `_lym`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned no report.

## Disputed

None.

## Deferred

None.

## Rulings

- **F7**, a `decision`-class finding the parent held.
  The parent's ruling (decision notice #2123) records the rows-only narrowing: row subscriptions only, no state subscriptions.
  It asks for no code change, and the code already builds rows only, so there was nothing to carry out.
  I did not flag the ruling to the parent, as it is neither mistaken nor in conflict with the code.
