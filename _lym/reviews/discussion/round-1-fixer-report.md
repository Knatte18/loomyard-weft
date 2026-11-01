# Fixer report — discussion, round 1

Every finding's premise held against the code (`internal/dobbyengine/starts.go`, `internal/battencli/cancelcmd.go`, `internal/battencli/land.go`, `internal/boardengine/board.go`, `task.go`, `taskguard.go`, `store.go`, `internal/boardcli/cli.go`, `internal/dobbyengine/landnotice.go`, `lym board profiles`).
`lym loom validate-discussion` exits ok after the changes.

## Changes

- F1 (fixed as proposed): D4's second hold bullet excludes a front task, so a re-run of `lym adopt` never holds the fix task; D4 acceptance bullet added.
  Twins brought in line after the self-check: Scope In, Auto-mode assumptions, `PATTERN-adopt-hold` and support-log Q4.
- F2 (fixed as proposed): D4 states the slot fact (`slotsTaken`, `startReady`, template default 3); D8 lets dobby start an open front task past `max_runs`, bounded by one open front task per worktree; D8 acceptance bullet with `max_runs` held runs.
- F3 (fixed as proposed): D8 defines `fronttask`'s removal (`lym batten cancel --remove`, record drop, entry removal) and states cancel's draft write and kept pair; a front task is closed once done, removed or in any other status such as draft, which lifts the start hold; D5's green end removes the fix task through it; D5 acceptance says no fix-task entry remains and dobby starts the released tasks.
- F4 (fixed as proposed): D7's `landed` reads an entry absent or done, citing Land's prune; Auto-mode assumption and support-log Q11 match; a hand-removed entry reading `landed` is stated.
- F5 (fixed as proposed): D6's slug uses `boardengine.MaxSlugLength` and appends six hex digits of the subject's SHA-256 when the subject is cut; the open-task test and D5's removal find the fix task by `front` field and project, never by slug; D6 acceptance covers two long subjects.
- F6 (fixed as proposed): D8 states that `fronttask`'s write carries no strand name, so `checkTaskCreation` does not judge it, and the bound (sole filer, closed caller set, parent's authorisation D3, creel #212).
- F7 (fixed as proposed, then narrowed by the self-check): `boardengine` refuses a changed `front` or `on_land` on `upsert`, `upsert-batch` and `merge`, dropping an equal value as `checkBeamPayload` does; the D8 acceptance bullet and the `PATTERN-refusal-way-forward` constraint name all three verbs.
- F8 (fixed with my own change): the proposed adopt-check-record recovery was not made. Instead `fronttask` keeps a machine-local record of each front task's action, which dobby runs when the entry reads done or is gone. This closes the gap for every front-task action, not only `adopt-check`, and keeps dobby from reading the adopt check's record. D8 acceptance covers a landing while dobby is down; the plan-writer note naming `notify_on_land` as the model is cut, since D8 now says why that memory cannot serve.
- F9 (fixed as proposed): D7 gains `halted`, ahead of `running_here`, naming the stop report's way forward; D7 acceptance covers it.
- F10 (fixed as proposed): D6 files the fix task with profile `light` and drops the "light today" aside; D6 acceptance names the profile.

## Self-check

- fix-consistency, MEDIUM, F1 exclusion missing from Scope, PATTERN-adopt-hold, Auto-mode and Q4: fixed as proposed.
- fix-consistency, LOW, "open" undefined for a front task: merged with fix-way-forward 1; fixed, D8 defines open as the empty or a run status.
- fix-consistency, LOW, D8 rationale counts three parts: fixed as proposed, it names the removal too.
- fix-claims, LOW, `slotsTaken` also skips a pawl run without its landing hold: fixed as proposed.
- fix-proposal, MEDIUM, refusing `front`/`on_land` outright rejects an entry sent back unchanged: fixed as proposed, only a changed value is refused.
- fix-proposal, MEDIUM, an unrelated open front task blocks the fix loop with no way forward: merged with fix-way-forward 2; fixed, the failed notice names a re-run of `lym adopt <subject>` once that task lands.
- fix-way-forward, MEDIUM, a draft fix task wedges later filings: fixed, a draft front task is closed (D8), D6 removes a draft fix task before filing, and D5's green end removes a halted or draft one.
- fix-way-forward, MEDIUM, notice for an unrelated front task names no way forward: merged above, fixed.
- fix-way-forward, MEDIUM, a failed removal after the record drop goes unreported: fixed, the removal orders cancel, record drop, then entry removal, so a failed cancel keeps the record; D5 reports a failed removal naming `lym batten cancel <slug> --remove` and `lym board remove <slug>`.
- fix-way-forward, MEDIUM, `halted` names the darn stop's `lym loom resume`, which makes no progress on a machine failure: fixed, D6's stop condition makes the stop report name the re-run of `lym adopt <subject>`, and D7's `halted` names it for a fix task.
- fix-way-forward, LOW, a filing that dies between board write and record leaves no notice: fixed with my own change, dobby records a front entry it sees open with no record, reading `on_land` from the entry, so the gap heals before dobby starts the task; the bound sentence is cut.
- fix-way-forward, LOW, the refusal names a reason, not a step: fixed as proposed, it names dropping both keys from the payload.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
