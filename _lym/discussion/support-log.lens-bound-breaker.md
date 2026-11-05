```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every claim, bound and guarantee in the drift-rework decision record.

**Findings**

1. **BLOCKING — D6, D7, D4 (runs in flight or past Drift-Expect).** D6 runs the seal check before every containment check, but D4 and D7 check a run with no `expected.yaml` against an empty expectation. A run past Drift-Expect when this lands has no seal, so the seal check fails before the empty-expectation path is reached.
   - D8 compounds it: the `drift` gate entry and `lym drift check` are "admitted only once the seal exists".
   - Counterexample: a darn run at Darn-Write when the binary lands has no seal. Its new `drift` gate entry either fails on every attempt, which leaves the writer Stuck with no way forward, or is refused. The record says neither.
   - Fix: state what a missing seal means. Either treat it as an empty, unsealed expectation that passes the seal check, or make it a Stuck that names a way forward.
2. **BLOCKING — D12 and D14 (`waive` removed).** A missing baseline that was not deferred has no way forward any more. The baseline capture lives only under `.lym/drift/`, which is never tracked.
   - Counterexample: `awaited_at` is set, then the run resumes on another machine or after `.lym` is cleaned. Drift-Verify finds no baseline directory. `defer` is admitted only at Drift-Await or a Drift-Baseline capture halt, and `waive`, today's verb for "no counted capture", is removed.
   - The same gap hits an in-flight run whose old baseline agent chose scope `off`. It took no capture and has no `awaited_at` deferral, while D3 now reads its config as `full`.
   - `PATTERN-durable-vs-ephemeral-state` says nothing a run depends on lives only under `.lym`. Containment now depends on exactly that, and the record is silent about it.
3. **MEDIUM — D1 (the stability claim).** "Keys keep a cell stable when a change inserts or removes lines" holds only for keyed lines.
   - Counterexample: a task inserts `roughness = 0.1` at line 3 of a `cs` case. Every unkeyed line below it, such as a header or blank line, shifts its absolute number. Each shifted pair is a changed cell outside the expectation unless the expect agent wrote an open range.
   - Pairing unkeyed lines by their position among unkeyed lines would keep the claim. Otherwise the record must narrow the claim.
4. **MEDIUM — D1 and D5 (repeated keys).** Occurrence numbering breaks stability, and a key entry covers too much.
   - Counterexample: a pretty-printed JSON case with many `"value":` lines. Inserting one item shifts every later occurrence, so the old and new values the judge sees are mispaired.
   - A `keys: [value]` entry covers every occurrence, so containment passes a regression in any other `value` of the case.
5. **MEDIUM — D6 (clearing a failed seal).** D6 says "only a parent amendment (D11) or a restored file clears it", but D11 "never edits `expected.yaml` or the seal". An amendment cannot clear a hash mismatch or a `Warp-SHA` mismatch.
   - "A restored file" names no actor or verb, which goes against `PATTERN-refusal-way-forward`.
   - A `Warp-SHA` mismatch can never be cleared at all.
6. **MEDIUM — D6 (restart path).** The seal takes two commits: the file is committed, then the seal is recorded. Today drift commits only at producer exit through `commitOnExit`.
   - Counterexample: a crash between the two commits leaves a committed `expected.yaml` with no seal. The record does not say whether a resumed Drift-Expect re-validates and seals the existing file or respawns the agent, which would rewrite it and invite a re-seal.
7. **MEDIUM — D8 and D7 (concurrent checks).** Concurrent checks collide on one scratch capture directory, and possibly on `capture.json`.
   - Webster's forks share the worktree and may each run `lym drift check` in parallel. Each writes "the scratch capture directory", which is one path.
   - D7 runs the capture "under the capture lock … as today's capture does". Today's `RunCapture` also records `capture.json`, which Drift-Await reads as the baseline's state, and moves an older baseline aside. The record needs to say the check uses its own directory and never touches `capture.json` or the baseline backup.
   - A check run while another fork is mid-edit captures a half-written tree. Its findings are advisory, so this is LOW on its own.
8. **MEDIUM — D8 (gate key under the plan's choice).** If the plan has a card or batch `gate.yaml` key run `lym drift check`, the check runs inside a held gate slot and asks for another.
   - `internal/gateslot` lets only a nested gate run, which reads `Pool.Inherited`, skip acquiring. Nothing says the drift capture's slot wait honours the inherited lease.
   - With one slot that is a deadlock. The record should require the check to run inside an inherited slot.
9. **LOW — D9 (judge tamper check).** "A judge session that leaves a tracked change outside `judgement.yaml` is a Stuck" fires on a tree that was already dirty before the judge spawned, for example from the writer or a tackler. Without a snapshot taken before the spawn, the judge is blamed for changes it did not make.
10. **LOW — D2.** "`compare` is required under `command` and refused under `lines` neither way" is self-contradictory. Nothing states a refusal for a missing `compare` under `command`.
11. **LOW — D2 (bound under-stated).** In `command` mode the judge sees only what the compare log shows, and the `cs` compare prints only the first deviating line per case. The original fault 2, a regression later in an already-drifting case, comes back. The bound names "gives up containment by code" but not this.
12. **LOW — D5.** Validation checks that cases exist but not that expected keys exist in the baseline case. A misspelled key is reported only as "covered no cell".
    - The default pattern keys `Re = 127069, dP = …` as `Re`, so a `dP` change is a `Re` cell. Nothing tells the expect agent the parsed key.
13. **LOW — D15 (run settings).** If `skip` becomes a retired key with `RetiredWayForward` in the module Spec, a board entry whose `run_settings` carries `drift.skip` is refused at seed. If it does not, the key is silently ignored. The record picks neither.
14. **LOW — D11.** It does not say whether amendment entries are validated by D5's parser, for example against an unknown case. Amend is admitted at a Stuck Drift-Expect, where no seal exists yet and an invalid `expected.yaml` cannot be fixed by an amendment.
15. **LOW — D6 (bound).** A cooperating-but-wrong writer can still forge a seal without rewriting history. It edits `expected.yaml` and `scope.json`'s seal fields before its first warp commit, and any weft commit made while the warp head still equals the baseline head then holds the edited bytes with a matching `Warp-SHA`. The bound says only "a rewrite of that history is outside it".
16. **LOW — Open risks.** The Parent-Align exposure is named for tapestry only. Darn's Finalize merges the parent after Drift-Recheck, so no compare covers that merge.

**Claims attacked**

| Claim | Result |
|---|---|
| D1 key stability | broken (3, 4) |
| D1 CRLF normalising | held |
| D2 bound | partly (10, 11) |
| D3 `docs` profile reaches existing hubs | held: profiles reconcile through `profiles.template-base` and gains new template keys |
| D3 `partial` read as `full` | held |
| D4 run past Drift-Expect reads an empty expectation | broken (1) |
| D4 Drift-Recheck Done when head unchanged | held |
| D4 no rename needed | held |
| D5 coverage | weak (4, 12) |
| D6 ordering via `Warp-SHA` | held, since no warp commit lands between branch point and Drift-Expect |
| D6 clear path | broken (5) |
| D6 restart | broken (6) |
| D6 forge bound | weak (15) |
| D7 lock and slot | broken (7) |
| D8 "decides nothing, writes only scratch" | broken (7, 8) |
| D9 tamper | weak (9) |
| D10 tuple reuse | held |
| D11 bound | weak (14) |
| D12 deferred path | held; missing-baseline path broken (2) |
| D15 run settings | open (13) |

**Tool calls made**

- Read `internal/gateslot/doc.go` for its nesting and inheritance rules, since my context had none on slot reentrancy.
- Grepped `internal/configreg/runsettings.go` for unknown and retired key refusal, and `internal/runprofile/doc.go` for how a template change reaches an existing hub. My context had neither.
