# Fixer report — discussion round 2

Every premise was checked against the code: `nowRFC3339` in `internal/shedengine/run.go` formats whole-second RFC 3339; `internal/shedengine/goto.go` persists `StatePaused`; `noticeCondition` in `internal/battenshed/notice.go` returns `noticeStateChanged` for every non-running state; `PRGate.Call` returns Done for a parent outside `RequirePRToBase`; `pushRebaseFreeLogged` in `internal/fabricengine/coalesce.go` Warns and returns nil on `ErrPushRejected`.
All five hold, and all five are fixed.

## Changes

- F1 (MEDIUM), fixed as proposed: D5 states `written_at` uses shed's RFC 3339 whole-second UTC form, the lower bound is strict and the upper inclusive, and a same-second verify failure fails safe to the blocked notice.
  Acceptance criterion added for a same-second genuine record and a same-second stale record from an earlier same-HEAD halt.
- F2 (MEDIUM), fixed as proposed: D5 suppresses the notice for the child's paused state at the target while the retry is unfinished; past the cap the notice names the child's actual state and the last refusal; the halted arm finishes an unfinished retry on every entry, not only after a restart, and the paused arm's resume on `goto-done` is named as the normal path.
  The rule-change sentence names both blocked and paused; the goto/resume acceptance criteria now cover the paused notice and the actual-state cap notice.
- F3 (MEDIUM), fixed as proposed (first option): D1 step 3 runs the deploy only for a parent in `require_pr_to_base`, skipping with an Info log otherwise; D2 says why and its Bound lists the condition; acceptance criterion added.
- F4 (MEDIUM), fixed as proposed: D6 states what each caller gets once attempts run out, Publish a Stuck with the push failure, the coalesced path today's Warn that leaves commits unpushed, told apart by a sentinel of its own; the push-retry acceptance criterion covers both.
  The rejected alternative (failing the coalesced push) is in the support log's Rejected alternatives.
- F5 (LOW), fixed as proposed: Scope now says the plain, never-forced pushes D6 names.
- Support log: round 2 entry under `## Review rounds`.

## Disputed

None.

## Deferred

None.
