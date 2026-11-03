---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/seatengine/engine.go:230
    summary: Engine.Run, Probe and Resume pass Geometry.DefaultEngine to Validate and startSeat passes the geometry to composeSeatPrompt, and no test drives them with DefaultEngine set
    fix: In internal/seatengine's engine tests, add one case that builds an Engine whose Geometry sets DefaultEngine and whose stencils dir holds a stencil including an engine block for that engine, then drives Run (and Probe or Resume) with a fake Shuttle and asserts the seat prompt carries the engine's block text; a second row with DefaultEngine empty asserts the refusal. This fails if any of these call sites passes an empty engine.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/burlerengine/engine.go:73
    summary: seatEngine's fallback (choice engine, else Geometry.DefaultEngine), Review passing opts.Review, Fix passing opts.Fix, and New handing DefaultEngine to the seat geometry have no test
    fix: In internal/burlerengine's engine tests, add a table test over Review and Fix with an Engine whose Geometry sets DefaultEngine, a RunOpts whose Review choice names one engine and whose Fix choice names none, and stencils whose instruction stencils include an engine block for each engine; assert the written instruction files carry the review choice's engine text for the review half and the default engine's text for the fix half. This fails on swapped opts, a dropped fallback, or a New that drops DefaultEngine.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/burlerengine/prompt.go:116
    summary: The explore step's own ReadSet call has no engine-block coverage; the engine-block test includes the block in the review and fix steps only
    fix: In TestComposePrompt_RendersEngineBlocksForTheHalfsEngine (internal/burlerengine/template_test.go), add burler-step-1-explore to the map of stencils that include the engine block and assert the explore instruction file carries the block text for the given engine.
---

### [MEDIUM] Seat engine entry points are unpinned

**Location:** internal/seatengine/engine.go:230, resume.go:30, resume.go:94, engine.go:293
**Issue:** The decision record's acceptance criterion asks that an engine block resolve for the seat's engine in composition and in the marker check.
`Table.Validate`, `CheckMarkers` and `Prompts` are pinned by new rows.
The entry points a real run takes, `Run`, `Probe`, `Resume` and `startSeat`, pass the geometry's `DefaultEngine` and are driven by no test that sets it.
A change passing `""` there compiles, passes every test, and refuses every engine-block table at run time.

### [MEDIUM] burlerengine's engine choice is unpinned

**Location:** internal/burlerengine/engine.go:56, engine.go:73-79, review.go:99, fix.go:157
**Issue:** The new behavior in `burlerengine` is the choice of the half's engine: the model choice's engine, else `Geometry.DefaultEngine`, handed to the instruction-file composition and to the seat geometry.
The new test calls `composeReviewPrompt` and `composeFixPrompt` with a literal engine string, so the fallback, the pairing of `opts.Review` with the review half and `opts.Fix` with the fix half, and the `New` copy of `DefaultEngine` are uncovered.

### [LOW] Explore step has no engine-block row

**Location:** internal/burlerengine/prompt.go:116
**Issue:** The explore step reads its blocks through its own `ReadSet` call.
`TestComposePrompt_RendersEngineBlocksForTheHalfsEngine` places the engine-block include in the review and fix steps only, so a regression in the explore call's engine argument goes unseen.

## Rejected

- lens:code-tests F4 (no test for the `DefaultEngine` wiring assignments): the wiring seams are reachable only through full CLI wiring, and the engine-level rows in F1 and F2 pin the behavior the assignments feed; a test per assignment would be a per-call-site test of a one-line field copy.
- lens:code-affected-set F1 (`battencli` and `landingdeps` `shedrecipe.Env` literals leave `DefaultEngine` empty): neither builds seats, no shipped stencil includes an engine block, and the lens itself found no present defect; it is not a change the diff owes.
- lens:code-affected-set F2 (`docs/shared-libs/stencil.md` not updated): the lens did not open the doc, and a search of it finds no include-resolution rule that engine blocks change.
- lens:code-correctness reported no findings; I found none in my own pass either. `stencilstore.ReadSet`, the `seatengine` and `burlerengine` threading, `PATTERN-engine-blocks` (landed with the declared text, with `PATTERN-fs-links` left to its own part), the three commit messages (no `Pattern-Departure` trailer) and the `codex-file` rendering match the decision record's D11 and D12.
