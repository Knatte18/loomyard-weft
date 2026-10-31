```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked**
- D6 entry path — "clean checkout and intact record end the row Done without starting or resuming a session": **broken** — a live session already present is never stopped (F1).
- D6 wake path — stop session, then check checkout and record, then end Done: held — the stop precedes the check; on dirty the session is resumed before it is mailed.
- D6 entry stamp — counts Done whose `on_done` names the row, or a goto from another row; halt/resume leave it: held — Goto/Relaunch append `OutcomeGoto` entries (`internal/shedengine/goto.go:135`, `:249`); no sampler row's `on_stuck`/`on_reroute` routes onto Sampler-Session.
- D6 Bound — "the verb is the only way the segment ends besides a halt or `lym batten cancel`": held — goto refuses a running status (`goto.go:226`), so a goto away always follows a halt.
- D6 Bound — flag accepted while halted ends the row at resume, even with no resumable session: held as stated; its unsafe interaction with a reopened session is F1.
- D4 — "row re-records the strand on every resume or restart, so the admitted strand is always the live one": **broken** by the record's own D6 Bound reopen paths (F2).
- D4 Bound — "each entry is … committed in its own diff, as the caller rule's package doc says": **broken** (wording, F4).
- D4 reader — "Describe and the Code-Review perch measure against the amended record": held — `landing-template-describe.md:23` and `landing-template-describe-darn.md:24` read `Added after Discussion` entries.
- D5 publish guard steer — "lym pushes the branch and opens any pull request only after `sampler-done` ends the row" / "steer text of its own": **broken** (F3).
- D3 — Finalize's `rerouteReviewed` and `codeBudgetZero` exist: held — `internal/loomcli/landingdeps.go:392`, `:401`.
- Constraints fact — empty landing w/o PR from commit `5f444b56`: held (commit subject matches).
- Constraints fact — watchdog activity sources, 60-min default, no row exemption: held against `agentActivityFor` (`looparm.go:132-186`) and `template.yaml:26`, whose comment counts trace writes as activity.
- D7 Bound — "a wedged step at a listed row is ended only by `lym batten cancel` or the row's own halts": held — goto cannot touch a running step; pause holds without ending.

**Findings**

- **F1 — MEDIUM — design — `decision-record.md` D6 entry path (lines 114-116) with its Bound (lines 135, 137).**
  Summary: the entry path ends the row Done on a current flag "without starting or resuming a session" and never stops a live session, so the agent can change the checkout after the check.
  Counterexample: run halted at Sampler-Session → the session is reopened (D6's own Bound: "`lym reed resume` may reopen the session", or the stop chain) → operator runs `lym loom sampler-done` from their shell (clean checkout, accepted while halted, stamps the current visit) → the reopened agent keeps editing/committing → `lym loom resume` re-enters, the entry path reads the flag *before it ensures a session*, finds a clean checkout, ends Done "without starting or resuming a session" — nothing stops the live session, which can then dirty or commit after the check while Describe and Finalize run, breaking the Bound's "Describe and Finalize always start from a committed tree". A second way in: the PR-Review shape D5 copies leaves the session running when the call's context is cancelled (`internal/landingshed/prsession.go`, `endSession`), so the row's next call after an interrupt meets the same live session. The entry path needs to stop a live recorded session before its read, as the wake path does at line 127.

- **F2 — LOW — design — `decision-record.md` D4 (lines 87-88) vs D6 Bound (line 137).**
  Summary: D4 promises "the admitted strand is always the live one", but only the row's own ensure re-records the strand; a session reopened by `lym reed resume` or the stop chain (admitted by D6's Bound) is not re-recorded by the row, so if its identity differs from the recorded one, its `decision add --by operator` is refused while the row is current, and if it does not differ, D4's re-record sentence is not what makes it hold. I did not verify whether `lym reed resume` keeps the strand name.

- **F3 — LOW — consistency — `decision-record.md` D5 fourth bullet (line 101).**
  Summary: the prescribed steer claims lym pushes the branch "only after `sampler-done` ends the row", but the guard's own doc says otherwise: `internal/shellguard/rows.go:247-248` — "A lym verb that pushes the task branch, such as lym fabric push or the detached both-sides push a run-record commit fires, reaches origin from the session too"; `internal/fabricengine/commitweftpaths.go:50` — `commitWeftAt` "fires a detached push unconditionally". (`decision add`'s own commit is the no-push `CommitAnchoredPaths`, `commitweftpaths.go:95`, via `commitRunRecord`/`commitDiscussion` `internal/loomcli/wiring.go:353`, so that verb does not break it; `lym fabric push`, unblocked by the guard, does.) Also, "steer text of its own" has no slot in the code: a row's `Steer` is the fixed constant `steerPublishDeny` (`rows.go:99`, `:251`) and the spec carries only `DenyPublish bool` (`internal/shuttleengine/spec.go:57`), so a sampler steer needs a new row or switch in the shellguard table (`PATTERN-shell-guard-table`), which D5 never says. The publish row also declares no `Forms`, so "the publish shell guard's blocked forms" names nothing the row has.

- **F4 — NIT — consistency — `decision-record.md` D4 Bound (line 92).**
  Summary: "each entry is … committed in its own diff, as the caller rule's package doc says" misquotes it: the doc says "in the call's one committed diff" (`internal/loomcli/decision.go:11`), and a `--from` batch commits several entries in one diff.

Tool calls:
1. grep `commitweftpaths.go` push behaviour, `rerouteReviewed`/`codeBudgetZero` definitions, commit `5f444b56` — to check D5's push claim and two cited facts the manifest lacked.
2. read `CommitAnchoredPaths` doc — whether a run-record commit pushes.
3. read `decision.go:740-762` — which commit `decision add` actually calls.
4. read `commitRunRecord` in `projectrecord.go` — one hop down that commit path.
5. read `commitDiscussion` in `wiring.go` — the commit a non-project record uses.
6. grep callers of the both-sides push — which run-record commits push (inconclusive).
