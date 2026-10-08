# Support log — gate-test-scope

## Interview

The session ran autonomously, so every question below was put and answered by the Discussion agent itself against the board entry, the related entries (`test-suite-weight`, `plan-gate-checks`, `publish-fix-route`, `verify-watchdog`, `convergence-plan`) and a bounded read of the code.

1. Scope of the card gate: tier 1 of the card's own packages, through which command?
   Read `cardGateCommand`: it emits `go build ./... && go test ./...` plus per-directory integration steps and the lint, and the fork runs it, not Go.
   Picked: keep the derived-command model and the nested-module split, but route the test step through a slotted `lyx gate test` so the fork's run counts against the hub bound (D1).
2. Where tier 2 runs once per batch.
   Go never runs fork gates; a Go-run batch gate inside `record-batch` would sit inside Master's Bash call and its timeout.
   Picked: a fork-run batch gate rendered beside the card gates, after the last card (D2).
3. How a fork's shell command takes a hub-wide slot.
   An OS lock taken by Go cannot cover a command the fork types; only a lyx verb can.
   Picked: `lyx gate test` as the one slotted route, and Go-side sites acquiring the same slots (D4, D7).
   A `lyx webster` verb was ruled out because the fork deny refuses every `lyx webster` command to forks.
4. Bash tool timeouts versus slot waits.
   A fork's Bash call dies at the tool's maximum timeout, so an unbounded wait inside `lyx gate test` could kill the call mid-test.
   Picked: a bounded CLI wait with a distinct exit and a re-run way forward; Go sites wait unbounded but visibly (D4, D6, D7).
5. Nested acquisition.
   A plan verify or card `**Verify:**` that itself calls `lyx gate test` would deadlock with every slot held by parents.
   Picked: an inheritance variable checked against a held lock, and the deny refusing any agent command naming it (D4, D7).
6. Which raw commands the deny refuses.
   The crash load was module-wide runs; package-scoped iteration is what forks need to work.
   Picked: refuse any `...` or `all` pattern on `go test/build/vet` and any raw `tmux`/`llm`-tagged `go test`; leave package-scoped tiers 1 and 2 raw (D7).
7. How a pair is kept from emptying `publish_verify`.
   `landing` is per-worktree and tracked, so a task's own commit could empty it and land the empty value.
   Picked: make `landing` hub-wide, reusing `ReconcileHubWideAt`'s `SeedPrime` (D10).
8. Tier 3 at the round gate, against its cost.
   Tier 3 started thousands of tmux servers in two days; most rounds touch no tmux-tier behavior.
   But `shuttle-send` blocked at Publish three times because no round ever ran the failing tier.
   Picked: tier 3 at the round gate only while a Publish failure record is present, plus the failing identities by name (D11).
9. The Burler range after a goto from Publish.
   After Publish merges the parent in, `merge-base` moves to the parent's tip, hiding the merged content.
   Picked: render the failure record and the merge-in commit's diff into Webster-Review's prompt, and make each failing identity a BLOCKING finding (D12).
10. Comment lint without a parser.
    Picked: a lexical opener-plus-verb heuristic biased toward flagging, because joining two lines always passes (D13).
11. Card `**Verify:**` under the deny.
    A plan card whose Verify is module-wide would be refused in the fork every time.
    Picked: a plan-gate check in the shared validation (D9).
12. The `llm` tier.
    Picked: stays by hand, recorded as a decision in `PATTERN-test-speed` (D14).

## Rejected alternatives

- **Slot acquired only in Go, card gate left as raw `go test`.**
  Does not bound the fork-run card gate at all, which was the crash's load.
- **Card and batch gates as a `lyx webster` verb that derives and runs the command.**
  Forks are denied every `lyx webster` command to prevent await-batch deadlock; carving out an exception weakens that guard.
- **Running the batch gate in Go inside `record-batch`.**
  It would run inside Master's Bash call and its timeout, and change Master's turn shape; a fork-run gate matches the existing card-gate model.
- **Denying every raw `go test`, routing all agent tests through `lyx gate test`.**
  Bounds everything, but queues a fork's seconds-long package iteration behind a Publish holding a slot for minutes; the board asks to refuse full-module runs, not all runs.
- **A FIFO queue or a slot daemon.**
  Ordering would need a daemon or a queue file with crash recovery; OS locks already release on death, and fairness was not asked for.
- **Unbounded `lyx gate test` wait.**
  Breaks against the Bash tool's maximum timeout.
- **Moving only `publish_verify` into a new hub-wide module.**
  Needs a cross-module key migration the reconcile machinery lacks, and leaves `require_pr_to_base` and `squash` equally pair-editable.
- **A Publish guard that refuses an empty pair `publish_verify` when the parent's is set.**
  Reads the parent's copy of a tracked file, catches only emptying (not weakening to `true`), and the parent copy is itself what the task's landing overwrites.
- **Impacted-set tier 3 at every round gate.**
  Costs tmux servers and minutes every round for a tier that rarely breaks; `test-suite-weight` exists because tier 3 is already heavy.
- **Never running tier 3 before Publish.**
  Leaves a fix round after a Publish failure with no evidence, the loop `shuttle-send` hit.
- **A POS-tagger library for the comment lint.**
  A new dependency for one heuristic, still imperfect on code-heavy prose.
- **Leaving card `**Verify:**` to stencil discipline alone.**
  A miss fails a whole batch under the deny; the plan gate check is cheap.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Card gate command shape — resolved, auto-pick: `lyx gate test` per module plus lint (D1).
- Q2 Where tier 2 runs per batch — resolved, auto-pick: fork-run batch gate after the last card (D2).
- Q3 How fork-typed commands take a slot — resolved, auto-pick: `lyx gate test` CLI verb (D7).
- Q4 Slot wait versus Bash timeout — resolved, auto-pick: CLI wait bounded by `cli_wait_sec` (default 300), Go-side unbounded and visible (D4, D5, D6).
- Q5 Nested acquisition — resolved, auto-pick: inheritance variable checked against a held lock (D4).
- Q6 Deny coverage — resolved, auto-pick: `...`/`all` patterns, raw `tmux`/`llm` tags, the inheritance variable (D7).
- Q7 `publish_verify` guard — resolved, auto-pick: `landing` hub-wide (D10).
- Q8 Tier 3 at the round gate — resolved, auto-pick: only while a Publish failure record is present (D11).
- Q9 Burler range after a Publish goto — resolved, auto-pick: render the failure record and merge-in diff into Webster-Review (D12).
- Q10 Comment lint rule — resolved, auto-pick: lexical opener-plus-verb heuristic biased toward flagging (D13).
- Q11 Card Verify under the deny — resolved, auto-pick: plan-gate check (D9).
- Q12 `llm` tier — resolved, auto-pick: by hand, recorded in PATTERN (D14).
- Q13 Defaults for `slots`, `go_parallel` — resolved, auto-pick: 2 and 4; operator-tunable.
- Q14 Overlap with `publish-fix-route` — resolved, auto-pick: this task delivers only the failure record, reviewer range and failure-driven round pass; the new fix route stays there.
