```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision traced on one concrete case, checking the end state against its stated purpose.

**Findings**

1. **MEDIUM, D1: the walk's starting point is not stated, and the plain reading resolves the wrong orch.**
   - D1 says the client resolves "from the caller's worktree with the same walk `orchcli.NotifyParent` uses (`hubgeom.NearestLiveOrch` over orch liveness)".
   - `NotifyParent` does not call `NearestLiveOrch` on the pair. `parentOrchStrand` first calls `hubgeom.ResolveParent(pair)`, then `worktreeOrPrime(pair, parent.Worktree)`, and only then `nearestOrchStrand(start, …)`.
   - Case: a beam pair that has its own live orch, whose agent runs `--parent`. `NearestLiveOrch(pair)` would test the pair's own orch first and could post to the agent's own worktree orch. That orch is not the parent, and not the orch the hold notice reaches.
   - D1's purpose of one resolver for the ask and the hold notice is met only if the walk starts at the parent worktree from `ResolveParent`. D1 and the D1/D3 acceptance bullet should name that start, or name `parentOrchStrand` itself.

2. **MEDIUM, D6: a program-signed post can be refused, depending on where the step process runs.**
   - D6 has the seat engine's lines posted "signed as the program of the module that runs the table".
   - creel refuses a program claim from a non-orch strand's pane. `internal/creelengine/identity.go:64` reads "only an orch strand's pane may claim a program name".
   - Case: an LLM-driver run. The driver strand launches `lym shed step --until-stop` as a background job from its own pane, and the step process runs the Discussion seat table. If that process is still in the driver pane's process tree, creel either refuses the program claim or the post fails.
   - Then the chair never gets "Advisor X is up". It waits, and its hold deferral never clears. Today's typed up line has no identity check, so this would be a regression.
   - The loom notifier already posts as `loomProgram` from the same process, so this may already hold or already be broken. Either way the record does not state it.
   - D6 needs the identity the post carries in both driver paths (detached loop versus dobby). It should also state the fallback when the claim is refused: a typed line, or signing as the driver strand.

3. **LOW, D10: an expected-reply message that is never read is never chased or marked `unanswered`.**
   - D10 starts the chase clock at the read.
   - Case: a pane-mail (Codex) receiver of agent mail gets only the reminder and never runs `lym creel read`. There is no chase and no `unanswered` outcome, so a `--wait-reply` sender waits until its own timeout.
   - This is within D10's stated purpose, which is a receiver that read the message and answered in its pane. The bound should still say that an unread message is left to the reminder backoff and the removed-holder sweep.

4. **LOW, D10: the `unanswered` notice cannot reach a program that has never read.**
   - D10's notice to the sender "unless EXTERNAL" includes program senders such as `LOOM`.
   - creel refuses a program receiver that has not read once ("A receiver is a strand name or the name of a program that has read once").
   - Case: a `LOOM` post with `--expect-reply` goes unanswered. The notice post is refused. D10 should state that a program's notice is dropped or journaled, as for EXTERNAL.

5. **LOW, D12/D11: the report's "expected replies" count needs the mark in the log, and D11 does not put it there.**
   - D12 counts expected replies through the Go client's log call. creel's log carries parties and subjects, no body, and no mark field.
   - D11 adds `chased` and `unanswered` events but does not say the posted event exposes the expected-reply mark.
   - Without it, `lym report replies` cannot count marked messages that got a reply and no chase. D11 should state that the log entry of a post carries the mark.

**Walkthrough list**

| Decision | Case | Purpose |
|---|---|---|
| D1 | Discussion writer in pair, beam orch dormant | Met only with the start corrected (finding 1) |
| D2 | Reply arrives from the prime orch after the D1 fallback | Met; the widened trust admits it |
| D3 | Held Webster Merriam, notice via `NotifyParent` | Met; same orch as D1 if finding 1 is fixed |
| D4 | LLM driver at an `awaiting` stop with `parent_notice`, watched | Met; `--parent` from a pair strand |
| D5 | Chair asks an advisor, advisor answers with `--reply` to the chair | Met; mail reaches the hook strand |
| D6 | LLM-driver Discussion table with advisors | At risk (finding 2) |
| D7 | Multi-seat table spec on Claude | Met; `claudeengine/capabilities.go` also lists the constant and must drop it |
| D8 | Operator interrupts a strand via `lym creel interrupt` | Met; no Go caller execs `lym shuttle send` (earlier grep found only comments) |
| D9 | Forward of a marked message | Met |
| D10 | Read, unanswered `--wait-reply` from the orch, then a reply to the chase id | Met; unread case and program sender gaps (findings 3, 4) |
| D11 | `log --id` of a chased message | Met for its events; the mark is not exposed (finding 5) |
| D12 | Fixture journal | Needs the mark in the log (finding 5) |
| D13 | Webster fork line in the allowlist; seat testdata outside `contracts/stencils` | Met |

**Tool calls**

- Read `parentOrchStrand` and `nearestOrchStrand` in `internal/orchcli/notify.go`, plus the creel program-claim rule in `identity.go`. My context had only `NotifyParent`'s doc line and not its walk start.
- Grepped the driver stencil for how the step loop is launched, and listed the `NewProgramClient` callers. My context did not show whether the step process runs under the driver's pane.
