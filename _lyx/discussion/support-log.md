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
   Round 3 found that resolving the `-C` dir through `preflight.ResolveMode` inside `internal/cliwire` breaks cliwire's import allowlist and refuses a nested module; `lyx gate`'s own pre-run now resolves the containing worktree (D7).
4. Bash tool timeouts versus slot waits.
   A fork's Bash call dies at the tool's maximum timeout, so an unbounded wait inside `lyx gate test` could kill the call mid-test.
   Picked: a bounded CLI wait with a distinct exit and a re-run way forward; Go sites wait unbounded but visibly (D4, D6, D7).
   Round 1 found that a chained multi-module gate could stack waits past the timeout; each gate step is now its own Bash call (D1, D8).
   Round 2 found that bounding the wait alone left the run under the timeout, and a killed `lyx gate test` would free its slot while `go test` ran on; every gate step now runs as a background Bash call, and `lyx gate test` kills its `go test` process group when terminated (D5, D7, D8).
   Round 3 found that card `**Verify:**` lines and verify-fix runs had neither rule, and that the kill had no `SIGKILL` or Windows story; both now run in the background with the same slot-busy handling, and `go` carries a Linux parent-death signal and a Windows kill-on-close job object (D2, D5, D7, D8).
5. Nested acquisition.
   A plan verify or card `**Verify:**` that itself calls `lyx gate test` would deadlock with every slot held by parents.
   Picked: an inheritance variable checked against a held lock, and the deny refusing any agent command naming it (D4, D7).
6. Which raw commands the deny refuses.
   The crash load was module-wide runs; package-scoped iteration is what forks need to work.
   Picked: refuse any `...` or `all` pattern on `go test/build/vet` and any raw `tmux`/`llm`-tagged `go test`; leave package-scoped tiers 1 and 2 raw (D7).
7. How a pair is kept from emptying `publish_verify`.
   `landing` is per-worktree and tracked, so a task's own commit could empty it and land the empty value.
   Picked: make `landing` hub-wide, reusing `ReconcileHubWideAt`'s `SeedPrime` (D10).
   Round 1 found that `lyx config <module> --set` on a hub-wide module from a pair writes and commits the hub file; picked a `configcli` refusal of hub-wide writes from a task pair (D10).
   Round 2 found that keying on the resolved worktree alone lets a pair's agent pass by running from the prime; the refusal now also keys on a slug-bearing `LYX_STRAND_NAME`, and D7's deny refuses commands naming that variable (D7, D10).
   Round 3 found that an env-clearing command drops the variable without naming it; D10's Bound and the Goal now state the refusal guards against a mistaken write, and name that route beside the direct file write.
8. Tier 3 at the round gate, against its cost.
   Tier 3 started thousands of tmux servers in two days; most rounds touch no tmux-tier behavior.
   But `shuttle-send` blocked at Publish three times because no round ever ran the failing tier.
   Picked: tier 3 at the round gate only while a Publish failure record is present, plus the failing identities by name (D11).
9. The Burler range after a goto from Publish.
   After Publish merges the parent in, `merge-base` moves to the parent's tip, hiding the merged content.
   Picked: render the failure record and the merge-in commit's diff into Webster-Review's prompt, and make each failing identity a BLOCKING finding (D12).
   Round 3 found that the editable record's merge field reached a shell command the reviewer runs; each field now passes a shape check before it renders (D11, D12).
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
  A stalled holder would leave the caller waiting with no named error and no holder list to act on.
- **Moving only `publish_verify` into a new hub-wide module.**
  Needs a cross-module key migration the reconcile machinery lacks, and leaves `require_pr_to_base` and `squash` equally pair-editable.
- **A standing deny on `lyx config` writes for spawned sessions other than the orch, in place of the `configcli` refusal.**
  Sees only static text, and would need its own carve-out for `--print`; the refusal keys on the resolved worktree and the caller's strand name, which a text match cannot.
- **Sizing `cli_wait_sec` against a stated run bound under the Bash timeout.**
  A module-wide integration run has no stable upper bound, so any split of the 600 seconds can kill a run mid-way; a background call is under no timeout.
- **Rendering only failure-record identities that also appear in the log.**
  The log path sits in the same editable record, so the check bounds nothing Publish's full rerun does not already bound.
- **Resolving `lyx gate test`'s `-C` dir through `preflight.ResolveMode` in `internal/cliwire`.**
  cliwire's import allowlist bars `preflight`, and `ResolveMode`'s anchor gate refuses every directory but the anchor, a nested module included.
- **Splitting `-C` into a worktree selector and a module dir.**
  `lyxcwd.ResolveWorktree` already finds the worktree from any directory inside it, so a second flag adds argument shape for nothing.
- **`go test` in `lyx gate test`'s own process group.**
  A group-directed `SIGKILL` would then take `go test` down too, but on a catchable signal `lyx gate test` could not kill the tree without signalling its caller's shell, which shares that group when job control is off.
- **One `lyx gate test` invocation holding a slot across every module of a gate.**
  Widens the verb's argument shape for a case per-step Bash calls already bound.
- **Keying the hub-wide write refusal on process ancestry or a reed marker instead of `LYX_STRAND_NAME`.**
  It would refuse an env-clearing `lyx config` from the prime, but a direct write to the hub's `landing.yaml` stays open whatever the refusal keys on, so it closes no route a determined agent needs; the refusal guards against a mistaken write.
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
- Q4 Slot wait versus Bash timeout — resolved, auto-pick: CLI wait bounded by `cli_wait_sec` (default 300), one background Bash call per gate step, card `**Verify:**` line and verify-fix `lyx gate test`, Go-side unbounded and visible (D1, D4, D5, D6, D8).
- Q5 Nested acquisition — resolved, auto-pick: inheritance variable checked against a held lock (D4).
- Q6 Deny coverage — resolved, auto-pick: `...`/`all` patterns, raw `tmux`/`llm` tags, the inheritance variable (D7).
- Q7 `publish_verify` guard — resolved, auto-pick: `landing` hub-wide, with hub-wide `lyx config` writes refused from a task pair or a slug-bearing strand, as a guard against a mistaken write (D10).
- Q8 Tier 3 at the round gate — resolved, auto-pick: only while a Publish failure record is present (D11).
- Q9 Burler range after a Publish goto — resolved, auto-pick: render the failure record and merge-in diff into Webster-Review (D12).
- Q10 Comment lint rule — resolved, auto-pick: lexical opener-plus-verb heuristic biased toward flagging (D13).
- Q11 Card Verify under the deny — resolved, auto-pick: plan-gate check (D9).
- Q12 `llm` tier — resolved, auto-pick: by hand, recorded in PATTERN (D14).
- Q13 Defaults for `slots`, `go_parallel` — resolved, auto-pick: 2 and 4; operator-tunable.
- Q14 Overlap with `publish-fix-route` — resolved, auto-pick: this task delivers only the failure record, reviewer range and failure-driven round pass; the new fix route stays there.
