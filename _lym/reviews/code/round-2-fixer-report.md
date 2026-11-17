# Fixer report — code round 2

## Changes

- F1 (MEDIUM): fixed as proposed.
  - `internal/loomengine/plan_test.go`: a `TestPlanTable` subtest puts the lead on `sol` with `plan_lenses` set and asserts the lead prompt carries `spawn_agent` and `wait_agent`, and neither `subagent_type: "fork"` nor `lym shuttle await-forks`.
  - `internal/loomengine/discussiontable_test.go`: the same assertion for a `sol` chair, run beside two Claude advisors so a producer that passed an advisor's engine fails it.
    The shared check is `requireCodexForkWording`.
  - `internal/webstercli/verbs_test.go`: `TestBeginBatchCmd_PromptFollowsMasterEngine` puts Master on codex while Recovery keeps the default engine, runs begin-batch, and reads the written fork prompt for `your developer instructions` and not `your system prompt`.
- F2 (LOW, `decision`): carried out under the parent decision `#1694` (see Rulings).
- F3 (LOW): fixed as proposed in `_lym/landing/summary.md`: the count and list now include `fork-message` (and `background-command`, from the F2 ruling), and the Codex vocabulary names `send_message`.
  It also states the allowlist's key shape instead of a tally of entries.
  The file sits under `_lym`, so it is edited in the working tree and never committed by this round.
- F4 (LOW): fixed as proposed in `cmd/lym/agentmessaging_test.go`: the `engineToolAllowed` doc, the `forkMessageBlock` constant and the rows `AllowlistedBlockLine` and `AnotherLineInTheBlock`.
  The scan's failure message now says "a Claude engine block's line" instead of "a ForkSubagents stencil line".
- F5 (LOW): fixed as proposed in the header of `contracts/stencils/webster/webster-template-master.md`.
- F6 (LOW): fixed as proposed in `contracts/stencils/engineterms_test.go`.
  `claudeLiteralAllowlist` is keyed by file, enclosing declaration and term.
  `goLiteralTermViolations` walks each declaration (`declarationUnits`) and reports the declaration in its message.
  The five entries and the `SpecAndGoLiteralScans` row follow; the row now also holds a second literal in the same file that no entry admits, and a method literal.
  Heddle's `overview.go` needed two entries (`placeTitle`, `placeReach`).

## Self-check

- fix-code-correctness, MEDIUM: the Codex `background-command` copy told Master to act on the exit status while the stencil two lines later says never to read the exit code. Fixed as proposed: the copy drops "and exit status".
- fix-code-correctness, LOW: the discussion row had no advisor to confuse with the chair. Fixed: the row uses two Claude advisors.
- fix-code-correctness, LOW: the Claude copy at stencil line 156 rendered "act on … notification, then act by these rungs". Fixed: the line now reads "its result goes through these rungs again".
- fix-consistency, MEDIUM: the Go cold way forward in `verifygatereport.go` told Master to run `fix-verify` as a backgrounded Bash command, which contradicts the Codex copy. Fixed: the text says "as the gate-failure section of your prompt says", the doc and test message follow.
- fix-consistency, LOW: the landing summary counted four allowlist entries. Fixed: it drops the tally and states the key shape.
- fix-consistency, LOW: `doc.go` comments described Master's backgrounded calls. Fixed for the passages that describe the stencil's wording (the recover-batch, fix-verify and fix-plan runs).
  Not changed: the comments about the awaited shell prefixes and `masterAwaitedShellPrefixes` in `runlevel.go`, which describe the Claude engine's own background-wake mechanism and stay true.
- fix-proposal, LOW: the F6 key stopped at the declaration, so a method name or a grouped spec could collide. Fixed with an own change inside F6: a method is keyed `Type.Name`, a grouped spec by every name.
  The function-level boundary stays, since it is what the review proposed and a finer key would name line numbers that go stale.
- fix-proposal, LOW: the F1 discussion row had no advisor. Duplicate of the correctness finding above; fixed once.
- fix-affected-set, LOW: the cold way forward text and its test. Duplicate of the consistency finding above; fixed once.
- No fork returned without a report.

## Disputed

None.

## Deferred

None.

## Rulings

- F2: the parent decision `#1694` (creel from `ly:LOOM`, recorded in the decision record under "Master's backgrounded commands get an engine block") rules that the passages of `webster-template-master.md` that run `recover-batch` or the `fix-verify` step as a backgrounded Bash command move into an engine block, with a Claude copy of today's wording, a Codex copy that runs the command in the foreground through its shell tool, and both registered in `contracts/stencils/stencils.go`. Carried out, no flag to the parent:
  - New block `engine-background-command` with `engine/claude/background-command.md` and `engine/codex/background-command.md`, registered in `stencils.go`.
  - The Master stencil pulls it in at the `recover-batch` rungs, the `fix-verify` step and the `fix-plan` step, and every other mention of a backgrounded call is reworded for any engine.
    Its line count is unchanged, so `MerriamBase` is unaffected.
  - The decision named the passages as `recover-batch` and `fix-verify`; `fix-plan` runs the same way and moved with them.
  - The fork-completion passages ("end your turn", "completion notification" of a fork) are not touched: they concern forks, not a backgrounded command, and the decision did not name them.
- No live Codex session ran the new copy; the test rows render it only.

## Gate repair

The gate refused five comments this round added, two wrapped at a fixed column and three of 120 characters or more:
`cmd/lym/agentmessaging_test.go`, `contracts/stencils/engineterms_test.go` (two), `internal/webstercli/verbs_test.go` and `internal/websterengine/verifygatereport.go`.
Each is now one sentence per line with no line at 120 characters.
Only comment text changed.
