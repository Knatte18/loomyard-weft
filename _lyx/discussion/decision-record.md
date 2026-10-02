# Decision record — webster-recovery

## Goal

A webster run recovers on its own from three states that today need a hand edit or a human: a re-begun batch whose fork landed nothing (#329), webster's own rewrite of a begun card (#330), and an integration regression (#332).

## Scope

In:

- #329: dispatch-time validation of a re-begun batch, at `begin-batch`, at `run` entry and in `lyx webster validate`.
- #330: every webster rewrite of the plan keeps the begun batches' recorded card hashes in step.
- #332a: the batch fork fixes a failing test its own card caused before it reports.
- #332b: an integration regression gets one automated fix attempt by a cold strand before the run escalates.
- Regression tests for each issue's "Seen" scenario, the `internal/websterengine` package doc, refusal-spec rows if any refusal changes, and deleting CLAUDE.md's TEMPORARY items for #329, #330 and #332.

Out:

- What a card verifies (repo-wide scans in a card's `verify:`, `-tags integration`, Plan-Write changes): board task `verify-publish-hardening` owns that.
- Master re-running integration tests after a rework fork: also `verify-publish-hardening`.
- Repairing state.json of runs begun before this lands.
- A grouping batcher, or any change to execution order or batch grouping.

## Decisions

### D1 — A begun, unrecorded batch is still pending for forthcoming targets (#329)

Decision: when dispatch validation scopes out begun cards, the Create targets and Rename New sides of every card in a begun batch with no terminal record still count as forthcoming, exactly as an unbegun card's do.
A later card's `Uses:` of such a target is then excluded from the status check, as it already is for a pending card's target.
The begun cards themselves stay out of the resolve pass, as today.
Terminal batches' targets do not count as forthcoming: their done-checks proved them present.

Rationale: an unrecorded batch is either partly landed or not landed at all, and in both cases its targets are real or on their way.
This needs no git inspection and holds for any batcher and any begin order.

Bound: this lets a `Uses:` of a target declared by a begun, non-terminal card pass dispatch validation unresolved.
It grants nothing an unbegun pending card does not already grant today, it covers only Create targets and Rename New sides, and the producing card's done-checks at `record-batch`/`recover-batch` still fail its batch when the target is missing.
A begun batch that never reaches a terminal record (abandoned, or the run ends or escalates first) never runs those done-checks, but it keeps the run from an outcome of done: the run-exit check that every batch carries a terminal done record refuses it.

### D2 — One scoping rule for all three validation sites (#329)

Decision: `begin-batch`, `run` entry and `lyx webster validate` scope dispatch validation the same way: begun cards out of the resolve pass, non-terminal begun cards' targets forthcoming per D1.
`validate` moves from terminal-only scoping to this rule.

Rationale: the Seen case had `validate` report the plan valid while `begin-batch` refused it.
One rule, computed in one place, makes the pre-flight verb answer what the run will answer.

Bound: `validate` stops resolving begun, non-terminal cards, so drift in such a card's own targets is no longer reported by any of the three sites.
That card was fully validated at its first `begin-batch`, its bytes stay pinned by its recorded `CardHashes` (checked by `record-batch`, recovery and `rebaseline`), and `record-batch`/`recover-batch` still run its done-checks and drift detection.

### D3 — Webster's own rewrites move begun card hashes in the same step (#330)

Decision: every sanctioned plan rewrite (handle canonicalization in dispatch validation, handle binding and exact-tier drift repair in `record-batch`/`recover-batch`) updates the recorded `CardHashes` of each begun card it changed, in the same restamp that adopts the rewrite into `PlanFingerprint`/`PlanFileHashes`.
`rebaseline` and the begun-card checks in `record-batch`/recovery keep comparing exact hashes.

Rationale: the rewrite sites already funnel through one restamp that runs after a foreign-edit check, so the hash update rides on a guard that already exists.
Exact-hash comparison stays simple and needs no canonicalization-aware diff.

Bound: this moves a begun card's recorded hash without an operator naming it.
It runs only at the sanctioned-rewrite restamps, never from `rebaseline`'s acceptance of an operator edit, and only in a call whose foreign-edit check passed before the rewrite (`PlanEditError`, or run entry's fingerprint compare).
That check is what refuses a foreign edit, including one followed by a sanctioned rewrite: the rewrite never runs while an unadopted edit is on disk, and `rebaseline` never adopts an edit to a begun card.
The update also moves only a card whose recorded begun hash equals the `PlanFileHashes` entry for that card's file before the rewrite (both are SHA-256 of the file's bytes), so a card an earlier untracked rewrite already moved keeps its old hash and stays refused.

### D4 — A fork fixes what its own card broke (#332a)

Decision: the fork implementer prompt treats any failing test the fork observes, in its gate or in any other test run it makes, as that card's gate failure when the card's change caused it.
The fork fixes it under the existing self-fix cap, and reports `status: FAILED` when the cap runs out.
Caused-by-the-card means the test exercises code or files the card changed, or a repo scan or enforcement test flags a file the card wrote.
A failure the card did not cause is still reported, not fixed.

Rationale: in the Seen case the forks saw both failures and moved on; the fix path already exists (the self-fix loop) and only the rule for what counts as the card's failure is missing.
What the fork must run stays `verify-publish-hardening`'s decision.

### D5 — One integration-fix strand before escalation (#332b)

Decision: when the run-exit integration stage triages a regression, it spawns one cold strand at the recovery role, with `recovery_timeout_min` as its timeout, before it escalates.
One attempt per run, with no new config key.
Its Go-rendered prompt names the regressing test identities with their output tails, the plan's `## verify:` command, and the bisected card's file as a hint when bisect localized one (standalone mode has no bisector, so the hint is optional).
The strand fixes at HEAD, commits, and writes a minimal report (`status`, `head_sha`).
Go then re-runs the verify in-process and triages again.
No regression left: the outcome is not demoted, and summary.md and the integration report record the fix commits and the identities they cleared.
A regression left, a `FAILED` report, a timeout, or a check failure: the run escalates as today, recording the pre-fix bisect result under the reserved `-1` record without bisecting again, with outcome stuck and the fix attempt's result added to summary.md.
Go never resets the branch on a failed attempt: the strand's commits stay, and summary.md and the escalation's way-forward name the pre-fix head and those commits so the operator decides whether to keep them.

Rationale: Master has already ended when the integration stage runs, so the fix cannot be an in-session fork; `run` already owns the reed/shuttle wiring and the recovery role, and `recover-batch` shows the cold-strand pattern.
The regressions in the Seen case were small and fully described by their failure tails.

Bound: this adds a spawn and commits to the task branch after the last batch without a human.
It is bounded by:

- the single attempt, recorded in state.json when spawned, so a resumed run never spends a second one; `run --fresh` starts a new run and resets it;
- the `recovery_timeout_min` timeout;
- Go accepting the strand's commits only when the commits from the pre-fix head to the report's `head_sha` are all non-merge, none touches the plan directory or anything under `_lyx`, and HEAD reconciles against `head_sha` under the existing merge-only rule; otherwise the attempt counts as failed;
- the post-fix triage deciding success, never the strand's own report.
  Triage sees only failures, so a test the strand deleted or skipped reads as cleared and a weakened one passes; the prompt forbids both, and summary.md names the fix commits and the identities they cleared for the review rows below to judge;
- the later loom review rows seeing the fix commits like any other branch commit.

The lease is not held across the strand's wait or the verify re-runs, matching the stage's existing phase split.

### D6 — The fix strand is reclaimable on resume (#332b)

Decision: the fix strand's GUID is recorded in state.json, and `run`'s entry-time reclaim stops it like any other recorded strand.

Rationale: a run that crashes during the fix must not leave a strand committing behind a fresh Master.

## Constraints

- `CONSTRAINTS.md` applies; `websterengine` is bound by the Told-Geometry Invariant (no `lyxcwd`, no fabric import) and the Refusal Way-Forward Invariant (any new or changed refusal gets its row in `contracts/specs/refusal-spec.md` and a reaching test in the same commit).
- Every prompt is Go-rendered from a stencil under `contracts/stencils/webster/`, read at call time (Stencil Ownership Invariant); the fix strand's prompt is a new stencil there.
- Agents run as interactive tmux sessions through reed, never `claude -p`.
- The state-mutation lease is never held across a long block (`AcquireStateMutation`'s contract).
- Every plan rewrite is re-baselined immediately after the call that performed it, before its findings are examined, and persisted even when the call fails (the package doc's staleness rule); D3 extends that same step.
- No change to fork report fields Master reads, to batching, or to sequencing.
- Docs land in the same commit: the package doc's sections on dispatch scoping, the staleness guard and the integration stage.

## Auto-mode assumptions

No operator answered; every Decision (D1–D6) is the Discussion producer's own pick.
D1 departs from the board's direction for #329; the board's Done-when (the Seen scenario passes) is met either way.

## Open risks

- D4's caused-by-the-card judgment is the fork's own; a fork can still misattribute a failure and move on, and D5 then catches it at the plan level.
- A run already carrying stale `CardHashes` when this deploys still hits the #330 refusal, and its TEMPORARY workaround is deleted by this task.
- The fix commits sit after the last card SHA, so a later bisect in the same run cannot attribute anything to them; with one attempt per run there is no later bisect.

## Acceptance criteria

- #329 Seen: card 2 creates a file, cards 3 and 6 Use it; `begin-batch 02` records the batch, the fork commits nothing, and a second `begin-batch 02` succeeds.
  `run` entry and `lyx webster validate` give the same answer on that state.
- #329: a re-begun batch whose fork landed its Create target still re-begins without `create-already-exists`.
- #330 Seen: batches 2–4 begun; a later sanctioned rewrite (canonicalization at a later `begin-batch`, and separately a `record-batch` binding) rewrites text in begun cards; the operator edits pending card 5 and `rebaseline --card 05` succeeds.
- #330: a foreign edit to a begun card is still refused by `rebaseline`, by `record-batch` and by recovery, including when a sanctioned rewrite ran after the foreign edit.
- #332a: the rendered fork prompt carries the caused-by-the-card rule tied to the self-fix cap (a stencil property test, like the master template's).
- #332b Seen: an integration report with regressions bisected to card 6; the fix strand (fake starter) commits a fix; the re-run triage passes; the outcome stays done and summary.md records the fix.
- #332b: the strand reports `FAILED`, times out, or leaves a regression: the run escalates with the `-1` record and outcome stuck, and summary.md records the attempt.
- #332b: a strand commit touching the plan directory or `_lyx`, or HEAD moving by anything but the strand's non-merge commits: the attempt fails and the run escalates.
- #332b: a resumed run whose state records a spent attempt escalates without a second spawn, and entry-time reclaim stops a live recorded fix strand.
- No regression or pre-existing/flaky-only verdict: no fix strand is spawned.
- CLAUDE.md's TEMPORARY items for #329, #330 and #332 are deleted.
- `go test ./...` passes.

## Notes for the plan writer

- #329: `begunCards` and `completedCards` in `internal/websterengine/beginbatch.go`, `webstercli`'s own `completedCards` in `internal/webstercli/validate.go`, and `createTargetSet`/`renameNewTargetSet` built from the pending plan only inside `planglyph`'s `resolvePass`.
  The forthcoming set probably needs a second card set passed into `planglyph.ValidateDispatch`.
- #330: `restampFingerprint` in `fingerprint.go` holds the pre-rewrite `PlanFileHashes` at the moment it restamps, which is the comparison D3's bound needs.
  `planparser.RewriteRefs` rewrites plan-wide, which is how a later batch's binding reaches begun cards.
- #332b: `runIntegrationStage` and `recordTriageResult` in `runlevel.go`; `recover-batch`'s spawn/await in `recoverbatch.go` is the cold-strand pattern; the merge-only head cross-check lives with `record-batch`/`recover-batch`.
- #332a: `contracts/stencils/webster/webster-body-implementer.md`, "Bounded self-fix, then stop".
