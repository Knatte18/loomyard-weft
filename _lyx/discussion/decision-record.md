# Decision record — plan-gate-fixes

## Goal

Stop the plan gate's `delete-before-reference` check from flooding plans with false blocking findings by matching a deleted member through the same typed reference resolution `caller-uncovered` uses, and make the Plan-Review reviewer check its proposed fixes against the plan gate before writing them and name the enclosing function for a glyph tied to a comment (issues #562 and #563).

## Scope

In:

- `internal/planglyph`: `delete-before-reference` for a deleted member glyph finds references through `caller-uncovered`'s reference computation, restricted to later cards' Edit regions.
  This covers package-level members as well as the methods the entry names, since one computation serves both; their matching moves from a bare-word regex to the import-path scan or the typed load.
- `internal/websterengine`: batch recording and `recover-batch`'s delete-referenced refusal count only blocking `delete-before-reference` findings as refusal reasons.
- `lyx loom validate-plan`: an `--overlay` flag that validates the plan with proposed card and overview contents read from stdin, in memory.
- `contracts/stencils/loom/loom-rubric-plan-review.md`: the reviewer validates proposed target changes with that overlay, and names the enclosing function for a comment-tied glyph.
- Docs in the same commit: `internal/planglyph/doc.go`, the `delete-before-reference` row in `contracts/specs/loom-plan-spec.md` (no longer textual; its informational arm), `validate-plan`'s help text and the `internal/loomcli` module doc, `contracts/specs/refusal-spec.md` for the overlay's refusals, and `pattern/PATTERN-gate-self-check-parity.md`'s mode list, naming `--overlay` as a self-check mode no gate runs.

Out:

- `caller-uncovered`'s own semantics, including its informational name-only arm.
- A package self glyph Delete target: its import-path match is unchanged, apart from being limited to Go files (Decision 3).
- The Plan-Fix fixer's stencil: the fixer already runs the plan gate and may apply any admissible fix.
- Overlaying a card insert, remove or renumber: those go through `lyx loom insert-card` and `remove-card`, and the overlay replaces existing files only (Decision 6).

## Decisions

1. **One reference computation serves both checks at the plan gates.**
   For each deleted member, the plan gates compute references once, through `caller-uncovered`'s typed load and scan, and partition them: a reference inside a later card's Edit region is `delete-before-reference`'s, every other uncovered one stays `caller-uncovered`'s.
   A reference inside a later card's Edit region is `delete-before-reference`'s even when some admissible target also covers that line, as today.
   Each plan-gate pass runs one type load shared by both checks, never two.
   The typed, partitioned form runs at `ValidateFormat`, `Validate`, `ValidateRework` and `ValidateFormatAfter` over its pending cards, all through `planGatePass`.
   A package self glyph Delete's import-path match (Decision 3) runs in the same `planGatePass` at the same entry points, over the same later Edit regions, and stays blocking.
   The resolve-backed pass those entry points share with `ValidateDispatch` no longer runs `delete-before-reference` at a plan gate, so `ValidateFormatAfter`, which runs `ValidateDispatch` over the done cards, reports each reference once, from the typed form.
   Rationale: the issue asks for matching "as `caller-uncovered` already does"; sharing the computation keeps the two checks from disagreeing on what a reference is and avoids a second `go list` per gate run.

2. **Severity follows resolution.**
   A typed reference, whose used object `isSubjectObject` accepts as the deleted member, inside a later Edit region is a blocking `delete-before-reference` finding.
   A scan-resolved package-level reference (bare in the declaring package, import-qualified elsewhere) is blocking too.
   A method reference the scan matches by name alone, where no type information answered for that identifier, is an informational `delete-before-reference` finding, one per deleted method naming every file and line, mirroring `caller-uncovered`'s informational arm.
   Bound: this downgrades the guard only where no type information exists — a failed or timed-out load, a file with a parse error, a nested module the root-module load skips, an untyped identifier, or a pass that runs no load (Decision 4); there a real method call surfaces as informational and is caught by the later card's verify build.
   A failed load is logged, as `caller-uncovered`'s already is.

3. **Only Go code is searched.**
   A reference is an identifier in a `.go` file's syntax or tokens, so comments, string literals and non-Go Edit targets such as a `.md` file never match.
   A package self glyph Delete keeps its quoted import-path match, searched only in `.go` Edit regions.
   Bound: a prose mention of a deleted symbol in a later card's Edit target is no longer flagged; it never breaks a build.
   A later Edit region in a `.go` file under a directory the go tool ignores, `testdata`, `vendor` or a name starting with `.` or `_`, is skipped as `caller-uncovered`'s candidate walk skips it; the go tool never compiles those files, so a reference there never breaks the package build or the later card's verify build.

4. **Dispatch and Webster's batch recording run the check scan-only.**
   `ValidateDispatch` and the exported `LaterDeleteReferences`, which `websterengine`'s batch recording reaches through `planindex.Index`, run no type load: they use the token scan alone, so a method match there is informational.
   Batch recording counts only blocking findings as refusal reasons and as the trigger for its delete-referenced way forward.
   `recover-batch`'s `ErrRecoveryDeleteReferenced` refusal, which reaches the same check, likewise fires on blocking `delete-before-reference` findings only.
   Bound: a name-only method reference no longer fails a Webster batch; the plan gate already judged the typed answer before the run, a deleted member that its batch removed no longer resolves and is skipped as today, and the later card's verify build catches a real call.
   Rationale: dispatch re-validates against a tree the run has changed and carries no loader, and a type load inside Webster's failure path would wait on a gate slot where it may already be holding one.

5. **`validate-plan --overlay` validates proposed contents of existing plan files, in memory.**
   With `--overlay`, the verb reads one JSON object from stdin mapping plan file names to their proposed full content; the accepted names are `00-overview.md` and the card files the on-disk Card Index lists.
   Every read of a plan file's content in the check path goes through a reader that answers from the overlay first and disk second: the parse through `planparser.ParsePlanFrom`, and the done-card comparison, so an overlaid done card is reported as edited exactly as on disk.
   Handle canonicalization writes nothing in overlay mode: `resolvePass`'s `CanonicalizeHandles` step applies its substitutions to the overlay reader's contents in memory instead of through `planparser.RewriteRefs` on disk, and the re-parse after canonicalization reads through the same overlay reader, never `planparser.ParsePlan` on disk.
   Bound: the in-memory substitution follows the on-disk rewrite's semantics exactly, so an overlay run's later checks see the same canonical handles the Plan-Write gate's run would leave on disk.
   The directory listing stays the disk's, which the overlay cannot change because it adds and removes no file.
   It runs the flag-less mode's check set, and is mutually exclusive with `--rework` and `--require-approved`, as those two already are with each other.
   Nothing is written to disk, and the overlay never changes the tree the glyphs resolve against, so it validates target-list, verify-command and card-text changes, never a proposed code change.
   Bound and refusals, each with a way forward, a `refusal-spec.md` row and a test (`PATTERN-refusal-way-forward`): stdin that is a terminal or empty, stdin that is not one JSON object, a duplicate name, a value that is not a string, and a name that is neither `00-overview.md` nor a card file the current worktree's on-disk Card Index lists, which rules out path separators, `..`, `amendments.md` and an orphan `.md` file, a card name the Card Index lists whose file is absent on disk, and `--overlay` combined with `--rework` or `--require-approved`.
   Rationale: the Plan-Review reviewer may write only its review file, so it cannot stage a scratch plan copy; stdin keeps it read-only, and `PATTERN-gate-self-check-parity` holds because the overlay run calls the same functions as the Plan-Write gate.

6. **The Plan-Review rubric makes the reviewer validate proposed target changes.**
   In `## Proposed fixes`: before writing a finding whose `fix` adds, drops, moves or pairs targets, or changes a verify command, the agent writing that review file runs `lyx loom validate-plan --overlay` once with every such fix applied together, with the Bash tool's timeout raised to cover a type load, and proposes no fix whose overlay run reports a blocking finding the baseline run does not.
   The baseline is `validate-plan --overlay` with `{}`, never the flag-less run, so every run the reviewer makes writes nothing, as its one-file write surface requires.
   When the combined run reports a new blocking finding, it re-runs with each fix overlaid alone.
   A fix whose solo run reports a new blocking finding is withheld, and that defect is reported with the constraining check named instead of the refused fix, as the rubric already prescribes for a defect with no admissible fix.
   Every new blocking finding of the combined run that no solo run reproduces is a joint conflict: the agent withholds the fixes that finding names by card and reports those defects together, the same way, whether or not some other fix failed its solo run.
   A fix that inserts, removes or renumbers cards is not overlaid; the rubric already routes it through `insert-card` and `remove-card`.
   The directive binds writing a finding's `fix`, not a Bouncer row's focus file.
   The rubric points at `validate-plan --help` for the overlay's format and never restates it (`PATTERN-producer-pointer-rule`).
   Bound: per review file that proposes a target change, one baseline run and one combined run, plus one solo run per proposed fix when the combined run reports a new blocking finding, each slotted.

7. **The Plan-Review rubric makes a comment-tied glyph name its enclosing function.**
   A finding whose proposed glyph stands for a comment or line it cites names the function or method whose body encloses that line, confirmed with `lyx quarry resolve` that the cited line lies inside the glyph's span, never the function after or near it.
   When `lyx quarry resolve` shows no member span holding the cited line, such as a free-standing comment between declarations, the finding names the file's self glyph instead of a neighbouring member.

## Constraints

- `PATTERN-sole-parsers`: the overlay is parsed into a plan only by `planparser`.
- `PATTERN-gate-self-check-parity`: the overlay run and the Plan-Write gate call the same functions.
- `PATTERN-gate-slot`: the plan gates' shared type load runs through the index's loader, slotted wherever the index is `NewSlottedIndex`.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: each overlay refusal names a way forward the agents' settings allow; a heredoc into `lyx` is not a denied in-place edit.
- `PATTERN-stencil-ownership`: the rubric edit goes into `contracts/stencils/loom/`, never into the hub's deployed copy.
- `PATTERN-test-speed`: a test that needs the real type load is integration-tier.
- `PATTERN-comment-line-breaks` and the markdown semantic-line-break rule for every doc and comment touched.

## Auto-mode assumptions

No operator answered; each pick is the writer's own and is logged in the support log's question ledger.
The picks with the widest effect are Decision 2's informational name-only arm, Decision 4's scan-only dispatch and Webster path, and Decision 5's replace-only stdin overlay.

## Open risks

- A reviewer may still skip the overlay run; the rubric cannot force it, and the fixer's own gate remains the backstop.
- The overlay run adds review time, which shifts the cost baseline `twill-plan-followups` measures Plan-Review rounds against.

## Acceptance criteria

- The entry's motivating shape passes: card N switches callers off a type, card N+1 deletes that type's methods, and later cards' Edit regions hold unrelated `.Run(` and `.Call(` calls and `Run` in comments; the plan gate reports no blocking `delete-before-reference` finding when the load answers.
- Deleting a method `T.Run` while a later card's Edit region calls it on a `T`, a `*T`, through an embedding type, and as a method value, yields a blocking finding for each line when the load answers.
- With a failing loader, the same method call yields one informational `delete-before-reference` finding naming the file and line, and no blocking one; so does a call in a region file with a parse error.
- Deleting a package-level function while a later card's Edit region calls it qualified from another package, or bare in its own package, yields a blocking finding with or without the load.
- A later card's `.md` Edit target that mentions the deleted name yields no finding.
- A package self glyph Delete whose quoted import path appears in a later card's `.md` Edit target yields no finding, while the same import in a later `.go` Edit region stays blocking.
- With a done card in the run record, the plan gate reports a later-region method call once, blocking when the load answers, and never also an informational duplicate.
- A later card whose Edit region holds the deleted member's own declaration yields no finding for that line.
- A reference inside a later Edit region is reported by `delete-before-reference` and never also by `caller-uncovered`, and one outside every admissible target is reported by `caller-uncovered` alone.
- The plan gate runs the type load at most once per pass with both checks active.
- `ValidateDispatch` and Webster's batch recording spawn no type load; a name-only method match there is informational, and batch recording neither lists it as a reason nor picks the delete-referenced way forward, while a blocking package-level reference still does.
  `recover-batch` neither refuses on a name-only method match nor lets a blocking package-level reference through.
- `validate-plan --overlay` with an overlay that replaces a card's target list reports the overlaid plan's findings and leaves every plan file on disk byte-identical; with `{}` it reports the same as the flag-less run.
- An overlay that adds a non-canonical plan handle leaves every plan file byte-identical, and its later checks see the overlaid card.
- An overlay that edits a done card's file is reported as a done-card edit, as the same edit on disk would be.
- `validate-plan --overlay` refuses, each with its way forward: stdin that is a terminal, empty stdin, stdin that is not one JSON object, a duplicate name, a non-string value, a name with a path separator or `..`, a non-`.md` name, a name with no file on disk, `amendments.md`, an orphan `.md` file the Card Index does not list, and a combination with `--rework` or `--require-approved`.
- The Plan-Review rubric carries the overlay-validation directive and the enclosing-function directive, and the stencil claim and rubric tests pass.

## Notes for the plan writer

- `internal/planglyph/deleteorder.go` holds today's textual matcher, called from `resolvePass`, which `ValidateDispatch` also runs; `callercoverage.go` and `typesload.go` hold the reference computation to share, reached through `planGatePass`, and `callerCoverageFindings` drops a deleted subject's reference inside a later Edit region via `insideLaterEditRegion`.
- Existing untagged tests in `deleteorder_test.go` that expect a blocking method match will see an informational one under the untagged tier's failing loader; typed method cases belong in an integration-tagged test.
- `planparser.ParsePlanFrom` already takes a reader; `checkIndexFileConsistency` lists `plan.Dir` from disk, and `websterengine.EditedDoneCards` reads card bytes from `planDir`, which is the read the overlay must reach.
- `CanonicalizeHandles` (`internal/planglyph/handle.go`) writes through `planparser.RewriteRefs`, which itself re-parses with `ParsePlan(planDir)`, and `resolvePass` reloads with `ParsePlan` after a rewrite; both are the writes and reads Decision 5's overlay mode replaces.
  `cardHeadingNumber` (`internal/planparser/validate.go`) also reads card files from disk.
- New top-level tests may push `planglyph`, `loomcli` or `websterengine` past their row in `cmd/lyx/testdata/test-budget.yaml`; a card that does lists that file under `Edit:`.
