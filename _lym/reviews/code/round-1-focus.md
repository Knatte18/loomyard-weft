---
round: 1
focus:
  - "internal/shuttleengine/promptengine.go, claudeengine.go and codexengine.go (Prepare): does each Prepare refuse a mismatched or unset prompt engine, naming both engines and the producer's model setting, and does the hand-written mark pass without refusal on every path (D4)?"
  - "internal/shuttleengine/engineset.go: does the new call return the default engine's name for an empty model choice and the resolved engine name otherwise, and does every producer take its prompt engine from the same model choice it launches with?"
  - "internal/shuttleengine/promptengine_enforcement_test.go: does the scan cover every production launch-spec literal that sets Prompt (attach probes skipped), reject a literal setting neither prompt engine nor hand-written mark, and reject the hand-written mark outside internal/shuttlecli?"
  - "internal/orchcli and the orch spec: does the orch launch pin the Claude engine and its prompt engine explicitly, so a Codex default engine never gives Codex the orch role?"
  - "contracts/stencils edit directive copies and internal/editdirective: does each engine copy carry every rule of the removed shared edit-directive.md (Go formatting through lym loom fmt, scratch-file rules, renames, other-worktree rule), is the shared stencil gone with no leftover reference, and does Directive take the engine?"
  - "Producers split into a prompt render function taking the engine name and needing no session: bouncer.go judgeCall and runSeedSpawn, websterengine, treadleengine, samplerengine, landingshed, frictionengine, mergeresolve; check each Exports claim holds and no producer still renders for a fixed engine."
  - "internal/shuttlecli/run.go: does lym shuttle run mark its spec hand-written from both inline --prompt and --prompt-file, with the manifest naming the prompt hand-written?"
  - "pattern/PATTERN-shuttle-provider-seam.md and PATTERN-stencil-ownership.md: does the diff land the edit-directive section change D3 requires, and do the commit messages carry any Pattern-Departure trailer that names a real entry?"
  - "Test economy: TestProducerSpecs_SkillsAndParentDirective, TestBouncerEntry_EnvJudgeFallback and the contained claudeengine Prepare tests listed in the redundancy report; compare each candidate's assertions with its covering tests, and check new assertions that pin stencil prose."
---

Focus set from the decision record's D3 and D4 and the committed diff's package spread.
