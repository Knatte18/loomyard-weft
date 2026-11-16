```yaml
lens: forced-edits
outcome: reported
```

Review of the plan files through the forced-edits lens.

**BLOCKING — card 1: three helper glyphs name the wrong package.**
`internal/websterengine#seedConflictRecord`, `#requireNoConflictRecord` and `#newFixPlanFixture` do not resolve.
quarry gives `internal/websterengine_test#seedConflictRecord`, `internal/websterengine_test#requireNoConflictRecord` and `internal/websterengine_test#newFixPlanFixture`.
Evidence: `lym quarry resolve` returned not_found for the first and third, and `lym quarry glyphs internal/websterengine` lists all three under `websterengine_test`.

**BLOCKING — card 17: the `NewWebsterProducer` re-sign misses four callers.**
- `internal/loomshed/gatefindings_test.go:142` in `internal/loomshed#TestBatcherRows_StuckSurfacesTheBatcherError`
- `internal/loomshed/cancellation_test.go:47` in `internal/loomshed#TestCancellation_RealProducersReturnErrorNotStuck`
- `internal/loomrecipe/shape_test.go:100` in `internal/loomrecipe#frictionReflectProducerType`, which calls `loomshed.NewWebsterProducer(...)` with the old arity

None of these is on card 17, so the build breaks (`caller-uncovered`).
The same loomrecipe shape table at `shape_test.go:100` maps the `Implementer` engine to Webster's producer type alone, and no card considers whether it should know Weaver's producer.

**BLOCKING — card 22: the plan commit leaves Weaver's archive deletions unstaged.**
Card 22 widens the `ArchiveWebster` seam to also move `_lym/weaver/` into the archive.
The Plan-Write rotation that calls that seam (`internal/loomshed/planwrite.go:150`) commits through `internal/loomcli#planCommitPathspecWith`.
That pathspec adds only `websterengine.DirRel()` (`internal/loomcli/wiring.go:150-155`), so the deletions under `_lym/weaver/` stay as tracked dirt after the commit.
Fix: card 22 lists `internal/loomcli#planCommitPathspecWith` and `internal/loomcli#TestPlanCommitPathspec`, and adds `weaverengine.DirRel()` under the same holds-file-or-tracked rule.

**MEDIUM — card 22: deleting `TestArchivedWebsterDirs` makes a kept test's reason stale.**
`internal/loomshed/interruptpolicy_test.go:49` carries `//testtiming:keep pins that every policy value is a declared constant, which TestArchivedWebsterDirs never asserts` above `internal/loomshed#TestInterruptPolicies_ClosedVocabulary`.
Card 22 deletes the test that reason names and does not list this test.

**MEDIUM — card 16: the reworded reason text breaks Webster's tests.**
Card 16 changes the reason texts to "plan conflict" with the engine label and edits `planConflictWayForward`.
Three assertions pin the current text, "the same webster plan conflict recurred":
- `internal/shedadapters/webster_test.go:180` and `:190`, in `TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters/webster_test.go:560`, in `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`

Card 16 lists these tests only under `Uses`, and card 1 edits them only for the path move.
Card 16 must list both tests under `Edit`, or it must keep the Webster wording.

**MEDIUM — card 18: two more seam-marked tests reach the `Env` set.**
Two tests besides the loomcli wiring tests carry `//lym:seam` markers for the `Env` set:
- `internal/battencli#TestWire_EverySeamFilled` (`internal/battencli/wire_seams_test.go:197` onward)
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam` (`internal/testkit/envkit/envkit_test.go:25` onward)

Card 18 adds `Env.WeaverDeps`, which holds `weaverengine#RunDeps` and `weaverengine#Geometry`, and `Env.ImplementerSeam`, which is `weaverengine#ActiveDeps`.
Card 18 lists both tests only under `Uses`.
The tier0 seam scan refuses a test reaching `envkit.NilSeams` whose markers miss a struct of the `Env` set, so both need the three new markers under `Edit`.

**MEDIUM — card 20: user-facing renumber text still names Webster.**
Card 20 rewords the bound to "the last card the implementer has begun" and moves the lease to the active implementer.
These places still say Webster, and no card lists them:
- The `Long` help of `internal/loomcli#loomCLI.insertCardCmd` (`renumber.go` near line 242) and `internal/loomcli#loomCLI.removeCardCmd` (`renumber.go:276`): "must come after the last card webster has begun".
- The `renumberDeps.acquireLease` field comment (`renumber.go:32`, "webster's state-mutation lease"). Card 20 edits `renumberDeps`, but its Intent never mentions this comment.
- `docs/overview.md:1025-1026`, which says the verbs bound at the last card webster has begun and hold webster's lease.

**LOW — `gatefixer`'s role docs go stale.**
Weaver's fixer reuses `gatefixer.RoleVerifyFixer`, but `internal/gatefixer/roles.go:10` and `internal/gatefixer/doc.go:7` say that role is webster's verify-gate fixer.
No card lists them; they belong on card 11 or card 6.

**LOW — `docs/overview.md`'s package tree omits the new packages.**
The tree (for example line 327) lists every `internal/` package with a line each.
Cards 2 and 15 create `internal/weaverengine/` and `internal/weavercli/` but add no entries.
Card 16 also leaves stale the shedadapters line, which names Webster as the only implementer adapter.

**LOW — card 1 leaves Webster's plan-fixer verb help naming the old path.**
`internal/webstercli/fixplan.go:50` and `:71` describe the record that begin-batch leaves.
If either names the scratch-dir path, card 1 must list `internal/webstercli#websterCLI.fixPlanCmd`.
I did not open that verb, so whether the text names the path is unconfirmed.

**LOW — the D13 seam leaves one reader of Webster's state.**
`internal/failuretrace/trace.go:332` (`NewTracer`) still loads Webster's state only.
A Weaver run's landing failure trace then reads "no webster state to trace against".
D13 does not list this reader, and no card mentions it.

Tool calls I made, each for something my context lacked:
1. grep for the reroute and renumber texts and for the callers of `NewWebsterProducer`, `arbiterRecipeDescription` and `ArchivedWebsterDirs`, plus the `Env` seam-marker files: to find assertions and callers outside the cards.
2. awk for the enclosing functions of those call sites, the seam markers in battencli and envkit, configreg's imports and the gatefixer role docs: to name exact members and check the role docs.
3. grep for the envkit test name, the PATTERN-cli-cobra tally, the `docs/overview.md` mentions and the stencil registry: to check fixtures and docs the new modules force.
4. `lym quarry resolve` on card glyphs: to check that each spelling resolves.
5. `lym quarry glyphs` on websterengine: to find the real spellings of the card 1 helpers.
