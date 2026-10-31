```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guarantees and claims about existing code. Five claims broke and two Decisions have smaller gaps.

**Claims attacked**
- D1, "the check reads row engines, edges and keys only; a recipe without `contract: loom` is not checked": held.
- D2, "the exception admits pawl alone; batten seeds no run `lym loom` addresses": held. `internal/battencli/wire.go:533` names exactly tapestry, darn and pawl, and a prime seed is `batten`.
- D2's refusal placement against D14: the record is silent on it, but it is not broken today (F7).
- D3, the peer recipe fallback "empty meaning `tapestry`": broken (F5).
- D4, the board body verbatim under `## Goal` with every other section empty: broken (F1).
- D4, "when a record already exists it leaves both files untouched and returns done", which is how it stays resume-safe: broken (F2).
- D4, `decision add` before Record-Write refuses with the existing missing-record refusal: held (`internal/loomcli/decision.go:99`).
- D5, "a record whose sections bar `## Goal` are empty passes": held. `internal/discussionparser/validate.go:63-64` has no empty-section finding.
- D6, "a no-commit handoff halts; the halt lets nothing through, since it stops the run before Drift-Verify": partly broken (F6).
- D7, "the record carries everything the snapshot did": held.
- D9, the second Drift-Verify pass and the in-flight resumes: held. `tapestry-recipe.yaml:386-551` already runs Drift-Verify after Parent-Align, and Finalize reroutes to Parent-Align.
- D10, the claim that Shed's step-1 read pre-empts four `CheckSeed` verdicts: held (`internal/loomengine/seed.go:32-50`).
- D10, the claim that "what goes" is only the freshness check and the stat race guard: broken, and the removal states no bound (F3).
- D11, "no run seed records `beam`": held, no hub seed matches. The `lym shed` refusal of an unknown recipe also held (`internal/shedcli/table.go` `lookup`).
- D12, a profile naming `loom.plan_*` still applies after the move: held. `PrepareRunSettings` applies `moveRunSettings`, and `loom` stays registered, so `retiredByKeyMove` never fires.
- D13, "refusing rather than ignoring keeps a run pinned … from silently running another", and "refused when the run starts": broken (F4).

**Findings**

F1, MEDIUM, design, D4: copying the board body verbatim under `## Goal` turns the body's own H2 headings into record sections.
- Board bodies use H2s; this task's own body has `## Seen`, `## Config split` and `## Open`.
- The parser finds sections by scanning `## ` lines: `internal/discussionparser/append.go:329`, `carryover.go:247,355` and `citations.go:52`.
- `validate.go:63-64` says an extra H2 is no violation, so no gate catches the problem.
- Counterexample: a body holding `## Open risks`, `## Decisions` or `## PATTERN changes`.
  - Go's carry-over writes into the body's `## Open risks` inside Goal.
  - `decision add` appends under the body's `## Decisions`.
  - A body `## PATTERN changes` list is parsed as the record's PATTERN changes.
- D4 does not say how the body is embedded. Fencing it would work, since the spec says fenced lines are not read; so would demoting its headings.

F2, MEDIUM, design, D4: the "a record already exists" test is not resume-safe.
- If Record-Write crashes after writing the record but before the support log or the commit, a resume finds the record, writes nothing and returns done.
- Darn-Write's `discussion` gate is now validate-only and commits nothing (D5). Darn-Write's own commit covers only the landing directory (`internal/loomcli/darnwiring.go:153-154`).
- Result: a missing support log fails the gate, and the writer is told never to edit the pair, so the gate exhausts and the run halts. An uncommitted record is never committed.
- The skip test should be a committed, valid pair, not a record that exists.

F3, MEDIUM, design, D10: D10 removes guards beyond the two it names and states no bound for what now passes.
- Beyond the two checks D10 names, `CheckSeed`'s fresh-start rule also refuses a seed whose history names a row other than the two Preflights, a non-null `product.start_sha`, or `pause_requested` (`contracts/specs/loom-status-spec.md:152-153`).
- These are the half-finished verdicts. Only deliberate goto re-entries are waived today (`internal/loomshed/tapestrypreflight.go` `waivesHalfFinished`).
- D10 never says what a half-finished seed reaching the entry row can now do, for example run Discussion-Write over an existing record.
- It never says what bounds this either, for example that only a goto re-entry can bring such a seed there.
- The rubric's attack-surface rule makes an unbounded guard removal a finding.

F4, MEDIUM, design, D13: retired keys in an already-seeded pair's local layer are dropped silently, and the refusal fires at a different point than D13 says.
- Seed-Child writes the profile block into the pair's local `loom.yaml` or `darn.yaml` (`internal/runprofile/doc.go`).
- Once the keys are retired, the existing retired-key rule admits a CategoryNone key and every load drops it (`internal/configengine/doc.go:75-77`).
- Counterexample: an in-flight run whose local layer sets `arbiter_max_spawns: 0` or a `tackler` silently takes `shed.yaml`'s values after landing. That is the "silently runs another" outcome D13's rationale rules out. `driver_kind` holds only because a seeded run keeps its recorded driver.
- "Refused when the run starts" is also misplaced. `PrepareRunSettings` runs at board writes and at Seed-Child (`internal/boardcli/doc.go:62`, `internal/battenshed/seamchild.go:150`), and profiles are checked on every `profiles.yaml` write. So the refusal fires there, which D13 and its Open-risks line never state.

F5, MEDIUM, design, D3: the peers fallback misreads a pawl peer on another machine.
- `boardRecipe` seeds pawl from `Kind == project`, not from the entry's `recipe` field (`internal/battencli/wire.go:522-527`).
- So a beam project's entry has an empty `recipe`, and D3's "empty meaning `tapestry`" reads that pawl peer against tapestry's rows.
- That repeats the misreport D3 exists to fix.

F6, LOW, design, D6: two gaps in the no-commit halt.
- Respawn case: the prior-work respawn continues from commits an earlier spawn left past the fork (`internal/loomshed/darnwrite.go:120-130`). A later spawn that hits the outdated-entry stop therefore has commits past the fork, does not halt, and goes on to Drift-Verify with a stop report as its description. This breaks "the halt lets nothing through".
- Ordering: the handoff gates run inside the session before any producer-side check (`darnwrite.go:84`). A stop report that `validate-description` refuses exhausts the description gate, so the halt's reason is the gate findings, not "the change description" D6 promises.

F7, LOW, consistency, D2 against D14: D2 does not say where the non-loom refusal lives.
- `lym shed`'s tapestry, darn and pawl entries arm through the same `loomcli.ArmAt` whose `resolveRecipe` D2 cites as its model (`internal/shedcli/table.go:49-68`).
- A refusal placed there also refuses `lym shed` verbs, which D14 keeps for every recipe. Nothing breaks today only because pawl is excepted.
- D2 should place the refusal at the `lym loom` command, not in the arm both CLIs share.

Tool calls:
1. grep of `internal/discussionparser` for H2 and empty-section handling, to test the D4/D5 claims; the manifest had no parser code.
2. `tapestry-recipe.yaml` row order and `internal/battencli/wire.go:515-532`, for the D9 edges and the entry-to-recipe mapping behind the D3 fallback.
3. grep of hub `.lym/shed` seeds for `beam` and of `internal/shedcli` for loomcli dispatch, for D11 and D2/D14.
4. `internal/shedcli/table.go:40-80` and `ArmAt`, to see whether `lym shed` arms through loomcli.
5. `internal/discussionparser/validate.go` lines on order and extra headings, to confirm an extra H2 is not a violation.
