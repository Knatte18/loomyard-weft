# Card 5 — parent-review-no-pass-at-cap

**Edit:**
- `internal/parentreview#GateConfig`
- `internal/parentreview#closures`
- `internal/parentreview#closures.gate`
- `internal/parentreview#closures.final`
- `internal/parentreview#TestApprovePasses`
- `internal/parentreview/doc.go`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolveGateEntry`
- `internal/shedrecipe#resolveParentReviewClosure` -> `func resolveParentReviewClosure(entry string, attempts int, env Env) (shuttleengine.Gate, shuttleengine.Gate, error)`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/loomcli#TestDiscussionWriteRow_GateListIsDiscussionThenParentReview`
- `contracts/recipes/loom-recipe.yaml`

**Delete:**
- `internal/parentreview#closures.logPassedAtCap`
- `internal/parentreview#newPassAtCapGates`
- `internal/parentreview#TestGate_PassAtCap_CapRejectGoesToWriterThenRewritePasses`
- `internal/parentreview#TestGate_PassAtCap_BelowCapStillOpensNextRound`
- `internal/parentreview#TestGate_PassAtCap_FreshGateAfterCapRejectPasses`
- `internal/parentreview#TestFinal_PassAtCap_ConsumedCapRejectPasses`
- `internal/parentreview#TestFinal_PassAtCap_UnconsumedCapRejectFailsTerminal`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewPassOnCapLetsRewriteThrough`
- `internal/loomcli#TestParentReviewExchange_RejectThenFixGoesOnToPerch`

**Create:**
- `internal/loomcli#TestParentReviewExchange_RewriteReviewedThenCapRejectHalts`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`
- `contracts/specs/refusal-spec.md`

**Intent:**
D10: every rewrite after a parent reject is reviewed, and the reject cap halts the run.
This reverses commit 518dc7400's pass-at-cap mode.

`internal/parentreview`:
- `GateConfig` loses `PassAtCap`, and `closures` loses `loggedPassAtCap`; `closures.logPassedAtCap` is deleted.
- `closures.gate`: a reject on the latest round at the cap fails terminal through `closures.terminal` with the cap's line, consumed or not.
  Below the cap nothing changes: an unconsumed reject is consumed and fails with its findings, and a consumed one begins the next round and opens its request.
- `closures.final`: a consumed reject fails terminal with the "re-spawns the writer, whose discussion then goes to the parent as the next round" line below the cap and the cap's line at it; an unconsumed reject keeps its current terminal line.
- The `gate.go` file header, the `closures.gate` and `closures.final` docs and `doc.go`'s cap paragraph drop the pass-at-cap clause: at the cap the gate halts the run `blocked` until the parent approves.
- Tests: `newPassAtCapGates` and the five PassAtCap tests go; `TestApprovePasses` drops its PassAtCap row.
  The cap's terminal halt stays covered by `TestGate_CapFailsTerminalNamingWayOut` and `TestFinal_RejectAtCapNamesWayOut`.

`internal/shedrecipe`:
- `resolveGateEntry` refuses a `parent-review` element that carries a `pass_on_cap` key, whatever its value, before resolving the closure:
  `shedrecipe: <entry>: config key "gates" element <i>: "pass_on_cap" is refused on "parent-review"; way forward: drop the key, and raise "attempts" for more review rounds`.
  The input that reaches it is a recipe row whose `parent-review` element carries `pass_on_cap: true` or `pass_on_cap: false`, which a developer editing the recipe writes; every other element's `pass_on_cap` is read as before.
- `resolveParentReviewClosure` drops its `passOnCap` parameter and the `pr.PassAtCap` assignment; its doc keeps the cap-equals-attempts reasoning and drops the sentence on the Cap-th failure re-prompting once more.
- `resolveGateSpec`'s doc says `parent-review`'s `attempts` is the reject cap and that `pass_on_cap` is refused on it.
- `TestResolveGateSpec_Rejections` builds each row over `parentReviewEnv(t)`, which extends `newTestEnv`, so no existing row changes.
  It gains rows for `pass_on_cap: true` and `pass_on_cap: false` on a `parent-review` element, each refusing naming the row and `pass_on_cap`, and each also asserting `way forward:`.

`contracts/recipes/loom-recipe.yaml`: the Discussion-Write row's `parent-review` element sets `attempts: 2` with no `pass_on_cap`.
Its comment says every rewrite after a reject is reviewed as the next round, `attempts` is the reject cap and 0 turns the review off, and the cap's reject halts the run `blocked` until `lyx loom review approve`.
`TestDiscussionWriteRow_GateListIsDiscussionThenParentReview` asserts `attempts` 2 and no `pass_on_cap` key, and its doc comment and `testtiming:keep` reason say so.

`TestParentReviewExchange_RewriteReviewedThenCapRejectHalts` replaces the deleted integration test in `parentreview_integration_test.go`, on the same `newPRFixture`:
arrival 1 opens round 1 and carries the delivery prompt; `review delivered` from the task worktree and `review reject <slug>` from the prime; arrival 2 fails with findings naming round 1's review;
arrival 3, after the rewrite, is pending carrying a prompt naming round 2's brief, with round 2 the latest; delivery and a second reject from the prime; arrival 4 fails terminal naming `lyx loom review approve`.
That script returns a failing gate outcome carrying the terminal findings as its reason, and the producer's Call returns Stuck.
With the status file written `blocked`, `review approve <slug>` from the prime exits 0 reporting `superseded`, and a second Call's script sees its first arrival pass with no new round.
The check that the round's request rode along in the discussion commits stays.

Specs:
- `shed-recipe-spec.md`'s `parent-review` paragraph drops the `pass_on_cap: true` sentence: the gate fails terminal at `attempts` rejected rounds, halting the run `blocked`, and a `pass_on_cap` key on the element is a construction error.
- `refusal-spec.md` gains a loom-section row, `recipe: pass_on_cap on parent-review`: a recipe's `gates` list carries `pass_on_cap` on a `parent-review` element when the row builds; correctness halt; drop the key, and raise `attempts` for more review rounds.

The `reject` verb's help and `review.go`'s header already say every rewrite after a reject is reviewed and the cap's reject halts the run `blocked`, so they stay as they are.
A run in flight across the binary change needs no code: its recorded reject reads as below the new cap.
The engine-level `pass_on_cap` of every other gate entry is unchanged.

**ImpactSummary:** removes the parent-review pass-at-cap mode from the gate, its resolver, the loom recipe and their tests; every other gate's engine pass_on_cap is unchanged.
