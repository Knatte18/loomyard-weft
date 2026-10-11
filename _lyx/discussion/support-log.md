# Support log — twb-twc-1

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the related entries (`fork-wait-noise`, `proc-child-lifetime`, `audit-reader-globs`, `verify-watchdog`, `discussion-premise-cites`, `glyph-practice-audit`) and issues #500, #505, #507, #509, #511, #517, #522.
2. Base check on shells: at `800a0e892` `background_shell_wait_min` already expires nothing (`5a8be0274`); the remaining fault is that a gated turn end with outputs present counts at once over unawaited shells, and a gate shell is unawaited.
   Corpus case `master-fork-running` shows a fork's leftover shell in Master's Stop payload, so the shells are the forks' backgrounded gate runs.
3. Base check on Master self-fix: shuttle's generic re-prompt says "fix every finding", contradicting the stencil at the moment Master reads the failure.
4. Base check on recovery: `loadRunWrites` audits only Master sessions and their forks, and a respawn replaces the record naming the earlier recovery's run; `recover-batch` returns `output.Ok` for every terminal digest.
5. Base check on Create checks: `lyx quarry resolve` answers `found` for `cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed`, and `ambiguous` for `internal/proc#StartTime`; quarry's `statusForMatches` declines build constraints by design. #511's root cause is the variant ambiguity of #500, not unexported names.
6. Lens review: six forks, all returned. Findings, merged across lenses:
   - pattern-fit BLOCKING, D2 timeout path ends `Done`: fixed; D2 states shuttle's `Done`, the verify entry not passing and webster's demotion to `stuck` naming the discarded gate shell, with an acceptance line.
   - scenarios BLOCKING, how a hold ends with no new Master turn: fixed; D2 adds the Master rung for stray notifications, the assumption that the fork's completion starts a turn, and the timeout path.
   - scenarios BLOCKING, recovery writes reaching contract-file evidence: fixed; D4 keeps recovery writes in their own group, read only by uncommitted classing and reset.
   - premises MEDIUM + scenarios MEDIUM, recovery session id source and restart: fixed in D4.
   - scenarios MEDIUM, unauditable recovery transcript: fixed; degrade to no evidence with a Warn log.
   - pattern-fit MEDIUM, D3 restating the fork prompt; scenarios LOW, line length: fixed; the re-prompt points at the template section.
   - pattern-fit MEDIUM, D5 friction note: fixed; `noteRefusals` writes it.
   - pattern-fit MEDIUM, D8 refusal-spec rows that do not exist: fixed; the claim is removed.
   - goal-scope MEDIUM, D5 vs the board's "failed": fixed; named as an assumption with the base evidence.
   - scenarios MEDIUM, D5 re-entrant calls: fixed.
   - goal-scope MEDIUM, D8 external test unit: fixed.
   - premises MEDIUM, `BindHandles` does not read status: fixed; removed from D7's list, moved to Notes, and the acceptance line pins the binding.
   - scenarios MEDIUM, D7 invalid input, enumeration bound, `_test` suffix: fixed.
   - scenarios MEDIUM, `lyx quarry resolve` exit on a variant set: added to Out and Open risks.
   - board-fit MEDIUM, `weaver` inherits D6 and D4: fixed in Open risks.
   - board-fit LOW + pattern-fit LOW, second segment splitter: fixed; D1 says why the audit classifier is not reused.
   - board-fit LOW, `fork-wait-noise` independence: fixed in Open risks.
   - board-fit LOW, kick-start consumers of D7: rejected; D7's rule is exported, and naming unbuilt consumers adds nothing the plan needs.
   - board-fit LOW, `proc-child-lifetime` as a first user: rejected; supporting context only.
   - premises LOW, D2 rationale overstating the fixer's role: fixed; the rationale now names the first gated arrival.
   - premises LOW, D1 whole operators: fixed.
   - premises LOW + goal-scope LOW, quarry evidence from HEAD: fixed; D8 cites the base `git grep` and marks the #511 cause as inferred.
   - goal-scope LOW, D7's Plan-time half: fixed; D7 says what is new and what is not knowable at Plan time.
   - goal-scope LOW, D6 final reply with no reader: fixed; removed.
   - goal-scope LOW, D4 wider than "by batch": rejected; bounded and defended in Rejected alternatives.
   - scenarios LOW, empty prefix: fixed; D1 refuses it.
   - scenarios LOW, tag spellings: fixed in D6.
   - scenarios LOW, reset widening untested: fixed; acceptance line added.
   - pattern-fit LOW, `ignore` and `_test` filename rules: fixed for `_test`; `ignore` is an ordinary tag under the rule.
   - leanness MEDIUM, rejected alternatives in Scope Out and in D2–D4: fixed; trimmed to boundaries and rationale.
   - leanness LOW, Goal repeating Scope; assumption and constraint repeating D6; wrong function in Notes; Rename-old split across Scope and D7: fixed.

## Rejected alternatives

- Stopping a running gate shell at the turn end and recording its result as discarded: loses a result the fork still waits on; holding is the board's first option.
- Exact-prefix awaited match with `lyx gate test`: misses the `cd … && lyx gate test` and piped shapes forks use.
- A webster-supplied matcher function on the spec instead of segment matching in shuttle: more surface for the same rule; the segment rule is generic.
- A lyx verb that spawns the verify fixer: a verb cannot start an in-session fork, and a separate strand loses Master's context.
- Second recovery from a clean tree with the first diff as reference: needs Go to stash or reset uncommitted work and to re-apply it, with a failure mode of lost work.
- Attribution limited to the current batch's recoveries: other batches' recovery sessions are equally the run's, and the run-wide set is simpler.
- Making quarry honour build constraints: quarry is a separate module and rejects a guessed build context by design.
- Treating every `ambiguous` answer as resolving at done-check: would pass a real duplicate declaration.
- Letting a Create handle reuse a name its own card deletes (#507's first remedy): unnecessary once a variant set can be edited and deleted by glyph.
- Running tmux-tagged steps in the recovery: the board asks to leave them to verify.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Hold the gate shell or stop it? Auto-pick: hold (D2), bounded by `master_timeout_min` and liveness. Resolved.
- Q2 Where does the "gate shell" recognition live? Auto-pick: segment-start match of the existing prefixes in shuttle, `lyx gate test` added by webster (D1, D2). Resolved.
- Q3 Stencil rule or verb for the fixer? Auto-pick: rule in the re-prompt and report header (D3). Resolved.
- Q4 Attribution or clean-tree restart? Auto-pick: attribution by every recovery session of the run (D4). Resolved.
- Q5 Which terminal statuses exit non-zero? Auto-pick: `stuck` and `dead`; `failed` already does (D5). Resolved.
- Q6 Do unexported or test-file members resolve? Checked: yes, both resolve `found` at base; the fix is D7 plus the clearer detail of D8. Resolved.
- Q7 Quarry change or loomyard rule? Auto-pick: loomyard planglyph rule (D7). Resolved.
- Q8 Include #507's create-reuse remedy? Auto-pick: out of scope (Scope). Resolved.
- Q9 Leave tmux-tagged steps to verify, and llm too? Auto-pick: yes for both (D6). Resolved.
