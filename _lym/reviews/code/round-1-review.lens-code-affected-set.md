```yaml
lens: code-affected-set
outcome: reported
```

I checked everything outside the diff's lines that its changes affect. Two members are left wrong: a shipped recipe comment and a doc paragraph both still say the check that cannot judge holds the run.

**What each change affects, and whether it is covered**

1. **Gate behaviour: exit 5 (`gateslot.RedFirstUnjudgedExit`) passes instead of holding** (`internal/loomshed/redfirstgate.go` `verify`, `holdsForCommitOrRecord`, `redFirstHoldLine`, `redFirstFailure`).
   - The doc comment on `NewRedFirstGate` and the doc comment on the `RedFirstGate` `Gate` field: covered. The field's "pending on a refusal the writer's next turn may not lift" still holds for exit 4.
   - `internal/loomshed/doc.go`: covered.
   - `internal/shedrecipe/doc.go`: covered.
   - The `gateslot.RedFirstUnjudgedExit` doc: covered.
   - The darn writer stencil `contracts/stencils/darn/darn-template-write.md`: covered.
   - `contracts/recipes/darn-recipe.yaml:137`, the red-first gate comment: **not covered**. It still says "A diff that adds no test, and a check that cannot judge the tests, hold the run".
   - `docs/overview.md:1113`: **not covered**. It still says "a diff with no test, or a check that cannot judge the tests, holds the handoff".
   - `docs/overview.md:997`: still correct; exit 5 is still its own exit code.
   - `contracts/specs/loom-discussion-spec.md:135-138`: still correct; it speaks only of the `No test seam` record.
   - `pattern/PATTERN-gate-slot.md:97-99`: still correct; it describes the site, not the exit handling.
   - Tests: `internal/loomshed/redfirstgate_test.go` is updated. No test in `shedrecipe`, `loomrecipe`, `battencli` or `gatecli/redfirst_test.go` asserts the exit-5 wording.
2. **The verb's exit-5 message** (`internal/gatecli/redfirst.go:129`).
   - Assertion in `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged`: covered.
   - `gatecli/doc.go:472-474`: covered.
   - Help text at `redfirst.go:73`, the `runRedFirst` doc at `redfirst.go:105` and `gatecli/doc.go:489`: still correct, since the exit code is unchanged.
   - `noSeamWayForward` still has its exit-4 caller at `redfirst.go:164`, so nothing is orphaned.
3. **The `redFirstHoldLine` and `redFirstFailure` exit-5 branches removed.** No other caller or reader remains (the earlier grep found `RedFirstUnjudgedExit` only at the sites listed). Covered.
4. **Repository rules.**
   - No PATTERN entry or background file describes the exit-5 hold. No declared PATTERN changes are listed, so nothing is owed there.
   - No CLAUDE.md or PATTERN rule names `docs/overview.md` as a doc that must land with this kind of change.
5. **Commit message.** The `Root-Cause:` trailer names `holdsForCommitOrRecord` counting exit 5 as a hold, which matches the removed `|| res.ExitCode == gateslot.RedFirstUnjudgedExit` in `redfirstgate.go`. There is no `Pattern-Departure:` or `Loosened-Check:` trailer and none is needed: the exit-5 rows changed expectations by design, which is not a loosened limit.

**Findings**

- **MEDIUM**, class consistency, at `contracts/recipes/darn-recipe.yaml:137`
  - Summary: the shipped darn recipe's red-first gate comment still says "a check that cannot judge the tests, hold the run … until a new commit or that record lands". The diff makes exit 5 pass, and it updated the same wording in `internal/shedrecipe/doc.go`.
  - Evidence: `contracts/recipes/darn-recipe.yaml:137-138` against diff 005 at `internal/shedrecipe/doc.go:63-66`.
- **LOW**, class consistency, at `docs/overview.md:1113`
  - Summary: the overview still says "a diff with no test, or a check that cannot judge the tests, holds the handoff without spending an attempt". After the diff, only a diff with no test holds.
  - Evidence: `docs/overview.md:1113` against `internal/loomshed/redfirstgate.go:216-219`.
  - No repository rule makes this doc land in the same diff, so it is not BLOCKING.

**Tool calls**

- Grepped `contracts`, `pattern`, `PATTERN.md`, `docs/overview.md`, `tools/sandbox`, `README.md` and non-test `internal` files for the hold and No-test-seam wording. The manifest had not searched the recipe YAML, the specs or other stencils.
- Read `contracts/recipes/darn-recipe.yaml:125-145`, `contracts/specs/loom-discussion-spec.md:128-140` and `pattern/PATTERN-gate-slot.md:90-105`, and listed the other test files that mention red-first. These are the readers the grep turned up, none of them in the manifest.
- Grepped those other test files and the stencils for the exit-5 wording, to confirm no other test or stencil asserts what the diff changed.
