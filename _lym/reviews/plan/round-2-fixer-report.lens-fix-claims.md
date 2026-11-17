```yaml
lens: fix-claims
outcome: reported
```

Task: check that every factual claim the fix added or rewrote holds against the code at HEAD.

no findings

The claims I checked all hold:
- `fixSeatValues` takes `p *Profile`, so `fixSeatValues(p, fixFilesIn(e.probeDirOf()))` type-checks inside `SeatTables(p *Profile, …)`. That matches `internal/burlerengine/prompt.go:84` and `fix.go:254`.
- `review_path`, `review_format_path` and `instruction_fix_path` are not reserved markers. The reserved list in `internal/seatengine/table.go:56-60` holds `edit_directive` and `parent_directive`. `composeSeatPrompt` fills through `stencil.FillWith` with `table.Optional` (`internal/seatengine/prompt.go:206`).
- `PrepareEach` can call `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)`, because `shuttleengine.Config` carries `Agents agentconfig.Config` (`internal/shuttleengine/config.go:78`). `Runner.start` makes the same call before `Prepare` (`internal/shuttleengine/run.go:575-580`).
- `addSkills` writes the layout that `codexengine` resolves: `<root>/<marketplace>/<plugin>/<version>/skills/<name>/SKILL.md`. A name without a colon resolves to nothing (`skillload.go:149-182`).
- Claude's manifest holds the webster fork deny only for a `ForkSubagents` spec. The deny's `installed` check returns `in.forkSubagents` (`claudeengine/settings.go:306`).
- `RequirementSupporter` is `Supports(req Requirement) bool` (`internal/shuttleengine/engine.go:204`). The gate can read the engine's answer with no spec.
- Language skills come from the hub's layered `language.yaml` through `languageconfig.Load` (`internal/languageconfig/config.go:62-68`). Burler hands them to a seat only when `p.LanguageSkills` is set (`internal/burlerengine/engine.go:31`), which matches the "names skills only through the language skills and `ConfigRole`" claim.

Tool calls:
- Grepped the seatengine reserved markers and `FillWith`, to check the fixer stencil's markers are filled from values; my context lacked `seatengine`.
- Grepped the `claudeengine` fork-deny and `RequirementSupporter` definitions, to check the fork-deny gate claims; my context lacked them.
- Grepped the `language.yaml` loaders and burler's `languageSkills`, to check the language-skill source claim.
- Read `languageconfig` `Load` and the start of the `claudeDenies` entry, to confirm the hub layer and the deny's position.
- Read the fork deny's `installed` condition, to confirm the deny is installed only for `ForkSubagents`.
