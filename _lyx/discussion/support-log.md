## Interview

Autonomous run: no operator answered; every pick below is the writer's, recorded in the question ledger.

1. Read the board entry `twb-ftb-1` and the entries touching Webster: `arbiter-fixes` (deviations list, no overlap), `weaver` (reuses recovery verbs; D5 and D6 change their behaviour, not their shape), `glyph-use-audit`, `enclose-kickstart`. `twb-fta-1` is the sibling bench arm of this same entry.
2. Read the cited GitHub issues #500, #505, #507, #509, #511, #517, #522 for the run evidence.
3. Premises were read at base `800a0e892` (bench instruction in the board entry); nothing from `1e79825ff` onward counts as landed.
   Item 1 at base: `wait.go` no longer expires a shell, but `countsAtOnce` counts a fresh gated turn end done over unawaited shells; Master awaits only `lyx webster recover-batch`. Picked an awaited `lyx gate ` prefix.
4. Item 2 at base: the stencil already says "Do NOT localize or fix the failure yourself", but shuttle's generic re-prompt says "read it, fix every finding". Picked a per-entry re-prompt text.
5. Item 3 at base: `loadRunWrites` audits Master sessions and their forks only; recovery strands are separate sessions, so their writes are never evidence. Picked run-wide attribution including recovery sessions.
6. Item 3b: `recover-batch` returns `output.Ok` on every terminal digest. Picked non-zero for a non-done digest with the same fields.
7. Item 4: the sandbox rule bans mutating `lyx fabric` verbs; the tmux tier is compiled by the plan verify and run at Publish; `claudeengine`'s raw-go deny allows package-scoped `go vet -tags tmux`.
8. Item 5: probed `lyx quarry resolve` (quarry `v0.2.1`, same pin at base and now): `cmd/testtiming#rusageStats` and a `_test.go` helper resolve `found`; `tmuxkit#tmuxAllowed` resolves `found` today only because a later fix made it one declaration; `internal/proc#StartTime` resolves `ambiguous`. Concluded #511 was ambiguity hidden by the detail text.
9. Item 6: picked a lyx-side build-variant predicate over a quarry change, applied in the status policy and DoneChecks.
10. Lens review. Procedure note: both files were written before the forks were spawned, against the stencil's order; the forks waited on the ready marker, so each reviewed the finished files. Every fork returned a report. Findings:
    - premises: `lyx gate build` does not exist (BLOCKING): fixed, confirmed at base `internal/gatecli/cli.go` registers only `test`; D7 now names the rendered steps instead of enumerating commands.
    - premises: D7 omits `lyx loom lint-pattern`: fixed by the same change; at base the card gate renders only `lyx gate test` and `lyx loom lint-comments`, so no list is pinned.
    - premises: a `tmux` step can only come from a card's `verify:` line: fixed in D7.
    - premises + goal-scope: D6's non-zero set contradicts the spec row 66 ladder and the failed case already exits non-zero: fixed; D6 now covers only a dead digest with `recovery_retry` false and reuses that row's wording.
    - premises: `statusFindings` is also called from `drift.go`: fixed in D9.
    - premises: D5 must not rely on `ShuttleRunDir`: fixed, D5 audits by session id.
    - premises: working tree's later refusal row already shows D10's shape: no action, post-base.
    - goal-scope + scenarios: D5's hard error on an unreadable recovery transcript recreates the stall: fixed, it now degrades with a naming line; a never-recorded id and a re-begin are covered.
    - goal-scope + scenarios: D5 weakens the reset guard unasked: fixed, the reset's own-path set is out of scope.
    - goal-scope: Plan-time flag reads as silent narrowing: fixed, D9 states how the board's flag is met.
    - goal-scope: summary.md should call out a `lyx gate` shell: rejected, its "Background shells at the run's end" section already lists every such shell, and D3's wording rides the warning and note.
    - goal-scope: Rename new-side detail unrequested: fixed by naming it in Scope as the same Create direction.
    - board-fit: GOOS implications (`android` ⇒ `linux`) make D8 unsafe: fixed, D8 uses `go/build`'s matching rules with implied GOOS and `unix`, and an acceptance criterion pins `x_linux.go`/`x_android.go`.
    - board-fit: provider wording in D4 against `llm-agnostic-engine`: fixed in Constraints.
    - board-fit: name `weaver` as a `recover-batch` caller: fixed in Open risks.
    - board-fit: `arbiter-fixes` edits an adjacent stencil: rejected, no overlap.
    - board-fit: #511 premise is inference: fixed, D10 says so.
    - pattern-fit: D7's raw `go vet` contradicts the implementer body and sits outside `gateslot`: fixed, D7 names no step for the `tmux` line.
    - pattern-fit: D10's refusal keeps a way forward that loops: fixed, D10 adds the plan-edit branch to the `card not done` row.
    - pattern-fit: D6's row is new and should reuse the ladder: fixed.
    - pattern-fit: D8 should take the worktree root from its caller: fixed.
    - pattern-fit: out-of-order procedure unrecorded: fixed, recorded here.
    - scenarios: `answerSymbols` hides a variant-set Delete from the delete-order guard: fixed in D9, with a criterion.
    - scenarios: D6 criteria for elapsed `--wait` and `batch_failed`: fixed.
    - scenarios: D1 criterion should cover the re-arrival: fixed.
    - scenarios: legacy `// +build` lines: fixed in D8.
    - leanness: rejected-alternative clauses in Auto-mode assumptions, D5's chokepoint sentence, the repeated #511 premise, the repeated `cd` risk, generic constraints, D9's narration, the bench framing in Goal: fixed, each cut or reduced to a pointer.

## Rejected alternatives

- Item 1, lyx stops the shell and records the result discarded: lyx has no pid for a provider shell (shuttle doc), so it cannot stop one reliably; holding is both possible and what the board prefers.
- Item 1, hold every unawaited shell: a stray watcher shell would then pin Master until its deadline; gate shells are the ones whose result matters.
- Item 1, substring match on `lyx gate` anywhere in the label: matches commands that only mention the text, such as a `grep`; a prefix plus D2's stencil line is narrower.
- Item 2, a verb that spawns the fixer: a fork is an Agent-tool call in Master's session; no CLI verb can create one.
- Item 2, a header line in the findings file as well: the re-prompt is the line Master reads first; a second copy adds nothing.
- Item 3, clean tree with the first attempt's diff as reference: discards work, needs a destructive step outside `fabricengine/destroy.go`, and loses nothing D5 keeps.
- Item 3, attribution limited to the same batch's recoveries: Master and fork writes are already attributed run-wide; one rule for every session is simpler.
- Item 3b, non-zero on every non-done digest: a stuck or retryable dead digest's next step is another `recover-batch`, so a non-zero exit there would misreport a normal step.
- Item 3, recovery writes in the reset's own-path set: would let a reset move over paths its foreign-dirty guard protects today, which no board item asks for.
- Item 3, an unreadable recovery transcript as an audit error: blocks the very recovery the item is meant to unstick.
- Item 4, compile a `tmux` line with `go vet -tags tmux`: contradicts the implementer body's rule against raw `go` for the `tmux` tier and runs outside a gate slot.
- Item 6, treat GOOS values as plain mutually exclusive tags: `android` builds `linux` files, so `x_linux.go` beside `x_android.go` would wrongly pass.
- Item 6, change quarry to honour build constraints: a separate repo and release; the predicate in lyx fixes every reader here.
- Item 6, require the variant set to be exhaustive (some file selected in every build): `proc_linux.go`/`proc_windows.go` is not exhaustive and is still correct code.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): hold or stop a gate shell? Hold via an awaited `lyx gate ` prefix (D1). Resolved.
- Q2 (auto-pick): rule placement or new verb for the verify fixer? Re-prompt text per gate entry (D4). Resolved.
- Q3 (auto-pick): batch attribution or clean restart? Run-wide attribution including recovery sessions (D5). Resolved.
- Q4 (auto-pick): which terminal digests exit non-zero? A dead digest with `recovery_retry` false, revised after lens review (D6). Resolved.
- Q5 (auto-pick): how a tmux-tagged step is handled in recovery? Not run by the strand, left to the plan's verify and Publish, revised after lens review (D7). Resolved.
- Q6 (auto-pick): quarry change or lyx predicate? Lyx predicate, Go-only (D8). Resolved.
- Q7 (auto-pick): what "flag at Plan time" means once DoneChecks accept the set? The same predicate judges Edit, Uses and Delete glyphs at Plan; a Create has nothing to judge before it exists (D9). Resolved.
- Q8 (auto-pick): include Rename old side and #507's delete-and-recreate? Out of scope; the board asks for neither. Resolved.
