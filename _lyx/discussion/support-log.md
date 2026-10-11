## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry `twb-twp-2` and the board list; `fork-wait-noise`, `proc-child-lifetime`, `discussion-premise-cites` and `weaver` touch the same modules.
   Read issues #505, #509, #517, #522, #511, #500 and #507.
2. Explored base `800a0e892`: shuttle's turn-end rule (`expiredTurnEnd`, `awaitedLabel`, `doc.go`), Webster's awaited prefix (`runlevel.go`), the gate re-prompt (`gate.go`), the Master template's verify section, `runwrites.go`, `recoverbatch.go` (engine and CLI), `state.go`'s `BatchState`, the recovery prefix and implementer body, `cardgate.go`, `donecheck.go`, `resolve.go`, `create.go`.
   Ran `lyx quarry resolve` on `cmd/testtiming#rusageStats`, `internal/testkit/tmuxkit#tmuxAllowed` and `internal/proc#StartTime`: the first two answer `found`, the third `ambiguous` over `proc_linux.go` and `proc_windows.go`; a `_test.go` helper also resolves.
3. Item 1: base no longer expires a shell, but a gated fresh run still counts a waiting turn end with only unawaited shells outstanding; the gate shells were unawaited and their labels were wrapped commands, so the awaited match moves to command segments (D1).
4. Item 2: the generic re-prompt says "fix every finding", the cause of Master's self-fix; a caller re-prompt on the gate entry (D2).
5. Item 3: recovery sessions are not in `recordedSessions`; record them on the batch and read them as run writes (D3); `recover-batch` reports terminal digests through `output.Ok` (D4).
6. Item 4: the fabric ban sits in the shared implementer body, so the clarification goes there (D5); the tmux-tier rule follows the plan spec.
7. Items 5 and 6: unexported and test-file members resolve; the failures are ambiguous answers over build variants; planglyph reads disjoint variants as one member (D6) and the done-check detail names the lookup (D7).
8. Lens review: all six forks returned.
   - leanness: D2's rejected-alternative sentence — fixed, cut. Auto-mode 1's "rather than" clause — fixed. D3/D7 incident narrative — fixed, trimmed. D1's base-record sentence — fixed, folded into the deadline bound. GOOS list premise sitting only in Open risks — fixed, moved into D6. Constraints docs enumeration — fixed, shortened.
   - board-fit: overlap with `fork-wait-noise` — fixed, D5 limited to two statements and one sentence, scope Out names it. `weaver` inheriting D3 — rejected, D3's rule applies to any recorded cold strand already, no renaming needed. Kick-start pack as a span reader — rejected, unbuilt. twb siblings — no action.
   - goal-scope: D4 lists only stuck and dead — fixed, every non-done status. D3 run-wide vs by-batch — fixed, stated as a deliberate choice with its bound. Reset widening — fixed, stated as following from the shared evidence. D6 Plan-time reading and D7 breadth — no change, already an auto-mode assumption and harmless.
   - pattern-fit: D4's refusal-spec row — fixed. D5 contradicting the body's tmux-tier sentence and calling tagged Verify steps allowed — fixed, D5 rewritten and the sentence reworded. `go vet` actor and scope — fixed, the plan's verify compiles. Shell-mechanics-seam line — fixed, added. D2 literal on the no-denied scan surface — fixed, noted. GOOS list source — fixed. One helper for variant reading — fixed.
   - premises: session id not on the start result — fixed, read from `run.json` or a new accessor. Contract evidence reads forks too — fixed. D6 "every reader" overreach and Rename — fixed, readers listed, Rename of a variant set refused and Out. `NewVerifyGate` location — fixed in D2 and Notes. D4 `dead` with `recovery_retry` — fixed. D7 premise breadth — fixed.
   - scenarios: missing recovery transcript (BLOCKING) — fixed, contributes no writes with a warning, criterion added. Master's turn on a gate shell's notification — fixed, template rule in D1. Re-prompt on attach and resume — fixed, criterion added. Rename old side — fixed (same as premises). Hung gate shell bound — fixed, deadline criterion added. Run-wide attribution — fixed (same as goal-scope). Other exit-code callers — fixed, plan confirms. Under-matching wrappers — fixed, named in the bound. Partial variant Create — fixed, Open risk. Empty-state criteria — fixed, added; missing-output-files case rejected as base behaviour already pinned by existing tests.

## Rejected alternatives

- D1: forbidding Master to run `lyx gate test` instead of holding — it leaves a disobeying Master's shell cut off; the hold is what the board asks for.
- D1: having lyx stop the shell and record a discarded result — lyx has no pid for a provider shell (shuttle `doc.go`), and the base already records every ended shell.
- D1: keeping a plain prefix and stripping a leading `cd … &&` — misses `|` pipelines and other wrappers.
- D2: a verb that spawns the fixer — only Master's Agent tool spawns a fork.
- D2: repeating the fork prompt line in the re-prompt — duplicates the template (PATTERN-producer-pointer-rule).
- D3: starting the second recovery from a clean tree with the first attempt's diff as reference — discards work and adds a reset path to a spawn verb.
- D3: attributing every uncommitted path to the batch — would claim operator or foreign edits.
- D6: changing quarry to honour build constraints — quarry is a pinned external module, out of this repo.
- D6: letting a Create reuse a name its card deletes (#507's first option) — D6 makes Edit and Delete by glyph work, which removes the need.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): hold the gate shell or stop it? Hold, via the awaited set and a segment match (D1).
- Q2 (auto-pick): re-prompt text or a fixer verb? Caller re-prompt on the gate entry (D2).
- Q3 (auto-pick): attribution by batch or clean-tree restart? Record recovery sessions as run write evidence (D3).
- Q4 (auto-pick): envelope shape for a non-zero `recover-batch` terminal exit? Error envelope carrying the digest fields (D4).
- Q5 (auto-pick): where the allowed-gate wording lives? The shared implementer body (D5).
- Q6 (auto-pick): what "flags such a glyph at Plan time" means? A variant set passes, a true ambiguity keeps `glyph-ambiguous` with the overlap named (D6).
- Q7 (resolved by reading): do unexported and test-file members resolve? Yes, so #511's fix is the message plus D6 (D7).
- Q8 (auto-pick): revive a dead batch through `goto` (#511 item 3)? Out of scope; the board entry does not ask for it.
