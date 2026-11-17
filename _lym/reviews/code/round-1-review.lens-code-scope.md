```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. Trace each change in the diff to the goal points and the Added-after-Discussion entries.

**The list: decisions and the changes that build them**

The answer key is a decision record alone. The decisions are the goal points under `## Goal` and the two Added-after-Discussion entries:

- **G1. Every stencil and spec passage naming a closed-list term sits in a Claude engine block with a Codex copy.**
  - Built by C2 and C3.
  - Specs need no change: `git grep` at the base `f8cc499c` finds no closed-list term in `contracts/specs`, and the spec scan in C6 holds that state.
- **G2. `mode_rules` and the plan-gate finding text are chosen per engine or worded for any engine.**
  - Built by C5a: `loomengine/prompt.go` `modeRules`, `planparser/verifymodule.go`, and the drift judge mode in `driftengine/judge.go`.
- **G3. The D2 scan.**
  - It reads stencils, specs and production Go literals, skips the orch places, admits allowlisted literals with a reason, and fails a block with no copy for a registered engine.
  - Built by C6 (`contracts/stencils/engineterms_test.go`), with the literal fixes in C5b.
- **Added after Discussion, 2026-11-16: the assist-trial schedule risk has no subject.** It asks for no code. No change, and none needed.
- **Added after Discussion, 2026-11-16: the round-3 findings are answered by D1, D4 and D12.** Those decisions belong to other parts of the beam, not this one. No change, and none needed here.

**The list: changes and the decision each traces to**

- **C1.** `stencilstore.ReadForEngine`. `ReadSet` now writes engine blocks in place for a plain include, and a copy is read without its final newline. Includes `doc.go` and `engineblock_test.go`.
  - Forced by C3: fragments filled with no include map, and blocks that sit inside a sentence, need it.
- **C2.** Eight new engine blocks, a Claude and a Codex copy each, plus their `stencils.go` embeds and registry rows. Traces to G1.
- **C3.** The stencil edits in burler, darn, gatefix, loom, raddle, seat, twill, weaver and webster. Traces to G1.
  - Some passages are reworded for any engine instead of moving into a block, for example "spawn no fork or subagent" and "each in full".
  - The goal point's outcome still holds, since such a passage no longer names a term, so this is not a scope finding.
- **C4.** Producers now read with `ReadForEngine` for the launch engine. Forced by C3: those stencils now carry block includes.
  - darn: `makeChangeList` and `DarnTable`.
  - `gatefixer.RenderPrompt`; weaver: `RenderBatchPrompt`.
  - `twilldirective`: `Values.Engine`, `fillPart` and `judgePart`; loomengine: `twill.go`, `discussiontable.go`, `plan.go`.
  - websterengine: `render.go`, `BeginDeps.Engine`, the `runlevel.go` reorder; `webstercli/beginbatch.go`.
- **C5a.** Go-filled agent text: `modeRules`, the drift `operatorJudgeMode`, the `verify-module-wide` detail. Traces to G2.
- **C5b.** Other production literals the scan would flag. Traces to G3; each is a literal the scan forces.
  - Fork-audit messages: burler `forkViolation`, webster `CheckFork`.
  - The twill fixer refusal "needs forks".
  - Help text: `shuttlecli/await.go` and `awaitforks.go`, `loomcli/parentreview.go` and `renumber.go`.
- **C6.** `engineterms_test.go`: the closed list, the orch skips, the allowlist, the missing-copy check over `modelspec.EngineNames()`. Traces to G3.
  - It also skips `internal/testkit/`, which D2's skip list does not name.
  - D2 limits the scan to production literals, and the repo's own taxonomy (PATTERN-test-only-code) counts testkit as test support, not production. So the skip stays within the decision.
  - A grep finds no closed-list term in testkit's non-test files today.
- **C7.** `stencilkit.SeedEngineBlockCopies` replaces `SeedEditDirectiveCopies`, and every test caller is updated. Forced by C3 and C4: fake engines now need every block.
- **C8.** `markers_test.go` drops `subagent_model` from the darn fragments. Forced by C3: the marker moved into the Claude block.
- **C9.** `MerriamBase` reads the raw Master template. Forced by C4: `MasterTemplate` now takes an engine.
- **C10.** Forced test updates:
  - `seatengine/table_test.go`: the lead's engine and the error text. Forced by C1 and C3.
  - `burlerengine` cluster, fix and smoke tests, `planparser` `validate_test.go`, `darnengine` `spec_test.go`, `loomengine` `plan_test.go`, `twilldirective` tests: forced by C5.
  - `websterengine` `export_test.go` and `template_test.go`: forced by C4.
- **C11.** `PATTERN.md` and `pattern/PATTERN-stencil-ownership.md`. Traces to the declared PATTERN change; it is the rule document C2, C3 and C6 make stale.
- **C12.** Comment and doc edits in `driftengine`, `loomengine`, `planparser`, `gatefixer`, `weaverengine`, `websterengine` and `twilldirective` `doc.go`. Forced by C4 and C5.

**Findings**

None. Every change traces to a goal point or is forced by a traced change. Every goal point is built by at least one change, and neither Added-after-Discussion entry asks this part for code. No commit carries a `Pattern-Departure:` trailer, and none is needed.

Tool calls:
1. Ran `git grep` for closed-list terms in `contracts/specs` at the base commit, and `grep` over testkit's non-test Go files. My context lacked whether the specs needed a change for G1, and whether the testkit skip excludes any live literal.
