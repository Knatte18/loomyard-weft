# Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically

`plan-glyph-precision` · Note · enhancement, loom, quarry

## Why

Seen on `gate-list`'s plan (2026-10-02): card 1's `Edit:` lists 24 file paths and no member glyph, although its Intent names the members it changes (`GateSpec`, `GateOutcome`, `Run.evaluateGate`, `resolveGateSpec`, `burlerengine#Engine.Run`).
Webster's changed-files guard then compares at file level only, no `quarry resolve` confirms the members exist, and a kick-start pack (`kick-start-pack`) has no spans to pre-resolve.

## Causes, in `contracts/stencils/loom/loom-template-plan.md`

- The only card skeleton shows `**Edit:**` with `` - `path/to/file.go` ``, and agents copy the shape they are shown.
- The glyph rule says a *symbol* target is spelled as a glyph and "a file path stays a plain path"; nothing says an `Edit` target should be the members changed, so a file path is always a legal choice.
- File paths are the path of least resistance: a member glyph must be looked up with `lyx quarry` and resolve at validation, and can fail as `glyph-not-found`, `glyph-ambiguous` or `containment-unit-overlap`; a file path only has to exist.
- `handle-unreferenced` requires every `plan:` handle to be used by a later card, so a new symbol only its own card uses cannot take a handle; the planner then invents a glyph (`internal/shuttleengine#GateEntry` under `Create:`), which the handle rule exists to prevent.

## Want

- The skeleton and the text say `Edit:` lists the member glyphs changed; a file path only for a file with no symbol to name (tests, non-Go files).
- The plan rubric flags an `Edit` card holding only file paths while its Intent names specific symbols.
- `handle-unreferenced` does not fire for a handle its own declaring card uses.

## First: test loomyard's use of glyphs systematically (operator, 2026-10-02, refined 2026-10-03)

The glyph library is part of quarry and the operator has tested it hard; what has failed is loomyard's own use of it (the plan stencil, the handle rules and the containment checks, in the sections below).
Before the stencil change pushes every plan onto member glyphs, test that use systematically against real plans: resolve every member glyph a sample of real cards would name, the handle bind path and the containment checks, and record what fails.

## Validation must not change verdict mid-run (2026-10-02)

Seen on `rework-generations`, batch 5: a plan valid at Plan-Write was refused at `begin-batch` with `containment-file-overlap`.
Card 4 creates `loomshed#ReworkTold` with a `plan:` handle; card 5 lists both `internal/loomshed/prrework.go` and `loomshed#ReworkTold`; card 9 lists `prrework.go`.
While the unit did not exist the handle had no location, so containment could not place it; once card 4 built it and `begin-batch` canonicalized the handle into a glyph, containment saw the overlap for the first time.
Cards 5 and 9 sit in different, sequential batches, so the overlap was harmless; the member line in card 5 was redundant beside its own file line.

Want: validation gives the same answer at Plan-Write and at dispatch.
Either containment places a `plan:` handle by its declared file at plan time, or it does not refuse overlaps between cards in different batches; and a card listing a file and a member of that same file is flagged at plan time as redundant.

## Mechanical caller coverage for deletions (seen 2026-10-03)

In `test-suite-change-amplification`'s plan the Burler found cards 36, 37 and 39 deleting helpers whose callers sit in files the cards do not list; plan-validate could not see it, because a card lists files, not the symbols it deletes or re-signs.
Once cards name deleted and re-signed members as glyphs, plan-validate can resolve every reference through quarry and flag a referencing file that no card in the same or an earlier batch edits, instead of leaving it to the Burler or to verify's compile failure.

## Priority (operator, 2026-10-06)

The operator wants this fixed soon: file-level targets waste tokens, since every agent downstream of the plan re-reads whole files to find the members a card changes.
It waits for `bugfix-webster`, whose Plan-Gate card-order checks change `internal/planglyph`.
The 2026-10-06 bugfix plans name targets by file on purpose, to survive the test prunes; that choice is not a precedent.
