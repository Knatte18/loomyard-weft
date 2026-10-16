---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: contracts/stencils/loom/loom-template-plan.md:210
    summary: the closing "See ... verify model section for the tier definitions" sentence appears twice in a row at the end of the verify paragraph
    fix: In contracts/stencils/loom/loom-template-plan.md delete line 210, the second of the two identical lines beginning "See `{{.specs_dir}}/loom/loom-plan-spec.md`'s verify model section for the tier definitions themselves", so the sentence appears once.
---

### [LOW] Duplicated pointer sentence in the plan stencil

**Location:** `contracts/stencils/loom/loom-template-plan.md:209-210`
**Issue:** Card 2 keeps the spec pointer for tier definitions once.
The diff leaves the original line and adds an identical one beneath it, so the rendered plan prompt tells the plan writer the same thing twice.
Both lines are byte-identical (`grep -n "verify model section" contracts/stencils/loom/loom-template-plan.md` returns 209 and 210).
No test catches it, because stencil static prose is exempt from assertions.

## Rejected

- Shared Decision "One marker value": every fill site (`PlanTable`, `ReadRubric`, `RenderForkPrompt`, `RenderRecoveryPrompt`) calls `languageconfig.VerifyTiersValue`, every `none` fallback sentence lives in a stencil, and no site is optional. No site builds the value another way.
- Per-card mechanical check: Cards 1–4 carry no `Rename` or `Delete` group. Every call site of the re-signed `Load`, `PlanTable`, `RenderForkPrompt`, `RenderRecoveryPrompt` and `ReadRubric` is updated; `battencli`'s `Env` is left unset as Card 3 says, and the unfilled-stencil readers (`twill.go`, `rework.go`) are left alone per the second Shared Decision.
- Stencil wording (Cards 2–6): the rules each card did not ask to change are intact. The dropped Go-tier examples and the "order the plan template states" sentence are the ones the cards name.
- Test economy: the `none` assertions in `TestPlanTable` and `TestRenderPrompts` match the list sentence ("this list gives:", "follow:"), which touches stencil prose, but the cards require them, so they are not raised.
  No new test duplicates an existing one, and the one new top-level test per package matches the `test-budget.yaml` rises.
- Comment conventions: the new doc comments in `internal/languageconfig`, `internal/loomengine/plan.go`, `internal/shedadapters`, `internal/shedrecipe`, `internal/webstercli` and `internal/websterengine` are self-contained and match the surrounding style.
- `loomCLI.language`, a struct field Card 1 does not name, is read by `TestWire_LanguageSkills`, as the card's test text requires.
- `lyx gate test` over the touched packages (`contracts/stencils`, `internal/languageconfig`, `internal/loomengine`, `internal/shedadapters`, `internal/shedrecipe`, `internal/loomcli`, `internal/webstercli`, `internal/websterengine`, `cmd/lyx`) passes.
