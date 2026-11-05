---
format: 5
approved: true
language: go
---

# Plan: pawl-orch-landing

A beam's landing work moves into Beam-Await, driven by the beam's orch with the operator: the orch writes the description, opens, updates, checks and closes the beam's pull request through new `lym beam` verbs, and `lym beam ready` becomes the point of no return.
Pawl drops Parent-Align, Beam-Describe and PR-Review and runs Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down; Finalize stays the only place a beam merges into its parent, lands through the pull request the beam's landing record names, and keeps its text.
Shed gains a recipe-declared retired-row map so an in-flight pawl run moves back to Beam-Await, and Beam-Await survives a loop exit and settles a landing hold it finds on entry.

## Card Index

1 — shed-retired-rows — shed moves a run off a recipe-declared retired row and pauses it
2 — landing-record — the tracked beam landing record beside the ready record
3 — pr-merge-state — read a pull request's merge state and required checks in one function
4 — describe-function — the beam description writer becomes a function
5 — pawl-rows — pawl's six rows, the Finalize hold, and the BeamDescribe producer's removal
6 — remove-reopen — remove lym beam reopen, the reopen request and PR-Review's reroute
7 — ready-beam-starts-no-member — the dobby and the member fork start no member under a ready beam
8 — retired-move-pawl — pawl's side effects of a move off a retired row
9 — await-survives-loop-exit — re-invoke policy for the pawl Go rows and the completed-wake repair rule
10 — beam-await-entry — Beam-Await settles a landing hold it finds on entry and stamps its wakes
11 — finalize-beam-record — Finalize lands a beam by its landing record and keeps its pull request's text
12 — beam-describe-verb — lym beam describe
13 — beam-pr-open-update — lym beam pr open and update
14 — beam-pr-status-close — lym beam pr status and close
15 — beam-ready-refusals — lym beam ready refuses what the orch can still fix
16 — member-landing-notice — the dobby announces member landings while a pull request is recorded
17 — beam-start-in-landing — lym beam start brings back the orch of a beam in landing
18 — orch-role-landing — the beam orch's role text carries the landing workflow

## Shared Decisions

- The landing record is `pawlengine.LandingRecord` at `loomengine.BeamLandingPath`, with its lock from `lock.TrackedFileLockPath`; every verb that writes it commits the landing directory in the beam worktree.
- The beam verbs share `landDeps` and its fakes in `newLandWorld`; `judgeAwaitingRun` and `judgeNotReady` (card 12) are the run-at-Beam-Await and not-yet-ready refusals every orch verb reuses.
- The retired rows are declared once, in `pawl-recipe.yaml`'s `retired` map; pawlengine reads their names through the told `Deps.RetiredRows`, since pawlengine imports no recipe package and loomshed imports pawlengine.
- `pawlengine` keeps its own `finalizeName` for the hold row, as it kept `parentAlignName`, since it must not import `loomshed`.
- `internal/loomcli/beamwiring.go` stays in loomcli, uncalled, from card 5 until card 12 replaces it with `fabricengine.BeamLineCommits` and beamcli's own description wiring; no dead-code check runs in the gates.
- A records-only commit, one whose every changed path lies under `_lym`, is lym's own bookkeeping and never a landing: the description (card 12) lists none and the member-landing notice (card 16) announces none.
- Beam-Await's entry finds an accepted landing by the board's hold, after a goto by the ready record standing with the await marker absent, or after a halted run's resume by the dobby-stop marker standing with no land request (card 10), since every shed transition rewrites the board status through the run's `CommitStatus` seam before the row runs; this is how D10's and D11's entry rule is carried.
- The beam verbs' tests and `ReadMergeState`'s drive GitHub through one shared fake, `internal/testkit/prkit` (card 3).
- `ReopenRequestFileName` and `Paths.ReopenRequest` stay after card 6 as the path of a reopen request an older binary wrote, which card 8's move removes.

## PATTERN changes

- `PATTERN-pull-request-mode` — changes: Deciding if a landing uses a pull request: only landingshed's one resolver, from the receiving `beam.pull_request`, the entry's `pull_request` or a beam's pull-request record, the parent's GitHub rules, origin and task commits; no run setting carries it; a yes merges on GitHub.
- `PATTERN-github-auth` — changes: Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize, `lym beam pr` and pair teardown open, reopen, merge or close a pull request; `lym beam pr` never merges. (test)
- `PATTERN-heddle-instructions` — changes: Rendering heddle entries into a file or prompt: only via `heddleengine`, from its shipped layer, Bolt's `heddle.json` and, under `lym_override`, lym overrides; `heddle.json` written only via `lym heddle` or its replayer; fabric names no heddle file.
- `PATTERN-owner-machine` — changes: Run start, resume, push or landing: only if no other machine owns its entry; a loom push reconciles run-scope origin first; owners only on roots and beams, set only by `boardengine`'s setter; others read their `hubgeom.ParentOf` owner; a dobby continue is no resume.
