# Decision record — small-fixes-batch

## Goal

Land six small, independent fixes in one task (board entry `small-fixes-batch`, issue #531):

1. The darn writer's prompt names a `lyx board get` form that works and a final-summary spec path that exists.
2. `docs/overview.md` states the recipes batten accepts as they are, and batten's terminal window opens once per batten run even when the child's status file vanishes mid-run.
3. `lyx config reconcile` raises a stored value its module's loader would raise to a floor, and reports the rewrite.
4. `lyx fabric unwire`, `lyx board retire-legacy` and `lyx board prune` change nothing without `--apply`.
5. The `board` group's command-index note prints only under the operator audience.
6. `WriteCarryOver` detects a concurrent writer of the decision record before and after its rename.

## Scope

In:

- `contracts/stencils/darn/darn-template-write.md` and `contracts/stencils/landing/landing-template-describe.md` (the same bare `lyx board get {{.slug}}` defect).
- `contracts/specs/final-summary-spec.md`, renamed to `contracts/specs/loom-final-summary-spec.md`, registered in `contracts/specs`'s registry, and every reference to it.
- `docs/overview.md`'s batten paragraph; `internal/battenshed`'s InnerRun terminal-opened marker and the batten fresh-status seeding in `internal/battencli`'s pre-run and pre-step hooks.
- `Migrate` hooks for the `shuttle`, `orch` and `loom` config modules in `internal/configreg`, implemented in their engines.
- The three destructive verbs, their engines' read-only dry-run reporting, and every caller lyx ships.
- `internal/clihelp`'s index-note rendering and `internal/boardcli`'s group annotation.
- `internal/discussionparser`'s `WriteCarryOver` and its refusal-spec row.
- The docs each change touches: package `doc.go` files, `docs/overview.md`, `contracts/specs/refusal-spec.md`, cobra `Long` text, `plugins/ly/skills/board/SKILL.md`, `tools/sandbox/*.md`.

Out:

- Floors whose value does not come from a config module: batten's `poll_interval_s` (a recipe row parameter) and shed's loop idle window.
- A lock shared by `AppendDecision` and `WriteCarryOver`.
- Any other no-argument verb beyond the three named.
- Changing the landing describe stencil's inline format list.

## Decisions

### D1 — darn prompt: JSON `board get`

**Decision:** `darn-template-write.md` and `landing-template-describe.md` both read the task with `lyx board get '{"slug":"{{.slug}}"}'`, the form `loom-template-discussion.md` already uses.
`internal/darnengine`'s stencil test pins the new form.

**Rationale:** `board get` takes a JSON object; the bare slug fails with `invalid json`, and the landing describe stencil carries the same defect.

### D2 — darn prompt: final-summary spec deployed beside the plan spec

**Decision:** Rename `contracts/specs/final-summary-spec.md` to `contracts/specs/loom-final-summary-spec.md`, embed it in `contracts/specs/specs.go` and register it as `loom-final-summary-spec`, so it deploys to `<specs_dir>/loom/loom-final-summary-spec.md`.
The darn stencil names `{{.specs_dir}}/loom/loom-final-summary-spec.md`.
Every reference to the old name moves to the new one: `docs/overview.md`'s links and spec list, `webster-spec.md`'s links and the landing describe stencil's mention.
The specs registry test lists the new name and its `loom/` placement.

**Rationale:** The board text proposes the repo path `contracts/specs/final-summary-spec.md`, which exists only in loomyard's own repository; darn runs against whatever repository a hub holds.
`PATTERN-stencil-ownership` reads a normative spec from a told absolute directory through `internal/stencilstore`, and `specs_dir` already reaches the darn stencil for this purpose.
The `loom-` prefix keeps the spec under the existing `loom/` family directory, which the registry test guards against one-file families; the final summary is validated by `lyx loom validate-description`, so loom owns it.

### D3 — overview.md recipe line

**Decision:** Rewrite the `docs/overview.md` batten sentence that says a board entry's `recipe` "must be `loom` or empty" so it reads `loom`, `darn` or empty, with `Seed-Child` refusing any other registered recipe, consistent with the paragraph's opening line and `internal/battenshed`'s `lyx loom start` serving both.

**Rationale:** The darn recipe has landed; the line contradicts the same paragraph and the code.

### D4 — batten terminal window: once per batten run

**Decision:** InnerRun no longer clears its terminal-opened marker when it finds no child status file.
The marker is cleared once, when batten seeds a fresh batten status for the slug, at both places batten seeds one: its pre-run hook and its pre-step hook, which seeds when absent so a fresh slug's first `lyx batten step` starts.
Both clear it through one function `internal/battenshed` exports beside the marker's declaration, so the marker name keeps one declarer.
A resumed batten run keeps the marker; a child whose status file vanishes and is re-spawned within the same batten run opens no second window.
`internal/battenshed/doc.go` and the InnerRun `Call` comment state the new reset point.

**Rationale:** The batten scratch directory is slug-scoped, so the marker must reset per batten run, and a fresh batten status is the one event that marks a new run.
The window runs `lyx reed attach` against the pair's reed session, which outlives a child re-spawn, so the first window stays valid.
Bound: the reset runs only where batten seeds a fresh status, so it clears the marker of a window already open only when the batten status file vanished mid-run while the scratch marker survived (risk R4).

### D5 — reconcile raises below-floor values through Migrate hooks

**Decision:** The `shuttle`, `orch` and `loom` config modules register a `Migrate` hook in `internal/configreg`.
Each hook rewrites a present file's key that its loader would raise to a floor to that floor value: `shuttle.poll_interval_ms`, `orch.poll_interval_ms` and `loom.step_idle_timeout_min`.
Each rewrite returns one report line naming the key, the old value and the floor; the existing `Migrated` field carries it on a dry run and an applied run alike.
Each hook reads its engine's existing floor constant, so the floor stays declared once; a value at or above the floor, or an absent key, is left untouched, and every other key, value and comment is carried through by yaml node editing, as `batcher.MigrateConfig` does.
A key with a null value (`poll_interval_ms:` or `~`) is raised to the floor and reported with old value `null`, because the loader decodes a null as 0 and raises it.
A key holding an `${env:...}` marker is left untouched with no report line, because its value resolves per environment and rewriting it would drop the operator's indirection; the loader still raises a resolved value below the floor and warns.
Any other value that does not decode as an integer, such as `abc`, is left untouched with no report line; the loader rejects it at load, as it does today.
None of these shapes makes the hook error, so reconcile goes on for every module.
The applied write is the template-shaped merge reconcile already produces from the hook's output.
`configreg.Module.Migrate`'s doc comment widens from "keys its module declares retired" to also cover a key raised to its module's floor.

**Rationale:** The observed fault is `poll_interval_ms: 500` kept under a floor of 1000, so every command warns.
The `Migrate` seam already runs before reconcile compares against the template and already feeds the report.
Bound: a hook only raises a value its own loader already raises at every load, so the effective configuration never changes; it never lowers a value and never touches another key.

### D6 — destructive no-argument verbs require `--apply`

**Decision:** `lyx fabric unwire`, `lyx board retire-legacy` and `lyx board prune` gain an `--apply` flag, the name `lyx fabric prune` and `lyx fabric cleanup` already use.
Without `--apply` the verb is a read-only dry run: it changes no file, junction, exclude line or board entry, exits 0, and prints the same envelope fields an applied run would, filled with what it would change, plus `"applied": false`; an applied run prints `"applied": true`.
`retire-legacy`, whose applied envelope carries no field today, gains a `legacy_files` field in both modes, listing the legacy files it folds and deletes or would.
A precondition refusal the applied run makes (e.g. `retire-legacy` with neither legacy file) is made by the dry run too.
`fabric unwire`'s dry run lists the junctions present and whether an exclude line would be removed; `board prune`'s lists the done slugs it would remove; `retire-legacy`'s lists the legacy files it would fold and delete.
`fabric unwire`'s dry run records no mutation, keeping `PATTERN-mutation-record`'s envelope shape.
Bare `fabric unwire` stays out of `fabricengine.FabricReaderVerbs`, so the fabric role guard still refuses it from a sandboxed role and `IsReadOnlyCommand` still classes it as mutating: no sandboxed role needs to probe unwire, and joining the reader set would widen the guard's and the read-only allow-list's surface beyond this task's ask.
Each verb's `Long` text and example state the dry-run default.
Every caller lyx ships passes `--apply` in the same change: `plugins/ly/skills/board/SKILL.md`, the board README's Done-section text in `internal/boardengine`, and the `tools/sandbox` suites that run `fabric unwire` (a sandbox step that checks the dry run says so explicitly); no Go code invokes these verbs.

**Rationale:** In the #512 incident bare calls of `fabric unwire` and `board retire-legacy` acted at once; a dry-run default makes a bare probe harmless and matches fabric's existing `--apply` idiom.
Bound: `--apply` restores today's behaviour, adding only the `applied` field and `retire-legacy`'s `legacy_files` report; the dry run reads state only.

### D7 — index note scoped by audience

**Decision:** `internal/clihelp` adds `IndexNoteAudienceAnnotation` (`lyx.index-note-audience`), whose value is one of `Audiences`.
The command index appends a command's index note only when rendering the audience that annotation names; a note without it never prints.
A test over the live command tree refuses a command that carries an index note without a valid note audience.
The `board` group names `operator`, the audience of every payload-taking board verb.
`internal/clihelp`'s and `internal/boardcli`'s doc comments state the rule.

**Rationale:** The board group's payload legend printed under the internal audience, where only `rerender` and `sync` list and neither takes a payload; a note scoped like a command's audience fixes it without guessing from children.
Bound: the change can only drop a note from an index, never add one.

### D8 — carry-over write checked against disk

**Decision:** `WriteCarryOver` re-reads the decision record immediately before its rename and refuses, writing nothing, when the bytes differ from the ones it read at its start.
After the rename it re-reads the record and returns an error when the on-disk bytes differ from the bytes it wrote.
Both errors name the decision record and say it changed during the carry-over write, by a concurrent writer such as `lyx loom decision add`.
The function's doc comment states both checks and that a post-rename mismatch leaves another writer's bytes in place.
`contracts/specs/refusal-spec.md`'s loom section gains a row, transient, for a carry-over write that found the record changed under it, with the way forward "run `lyx loom start` to re-run the settle once the other writer is done"; the settle's write is deterministic, so a re-run writes the same entry.

**Rationale:** A post-rename read alone only catches a writer that lands after the rename, which clobbers the carry-over; the pre-rename compare catches the writer that would otherwise be clobbered by it.
`lyx loom decision add` writes the same file without a shared lock.
Bound: the checks turn a lost update into an error in most interleavings, not all; without a lock two windows stay open (risk R2).

## Constraints

- `PATTERN-stencil-ownership`: the final-summary spec reaches the darn writer only as a deployed spec under the told `specs_dir`, through `contracts/specs`'s registry.
- `PATTERN-producer-pointer-rule`: the darn stencil points at the spec and does not restate its format.
- `PATTERN-mutation-record`: `fabric unwire`'s dry run keeps the `MutationRecord` envelope and records no mutation.
- `PATTERN-cli-cobra`: the new flags are cobra flags with JSON errors; each verb's audience is unchanged.
- `PATTERN-refusal-way-forward`: D8's new refusal lands with its refusal-spec row in the same commit and a test reaches it.
- `PATTERN-no-denied-recovery`: the D8 way forward names `lyx loom start`, which agents may run.
- `PATTERN-lyxdirs-single-declarer`: no new production file spells `_lyx` or `.lyx`.
- `PATTERN-test-economy` and `PATTERN-test-speed`: each behavior is pinned at its package's public surface, extending existing tables where one fits; filesystem tests stay in the unit tier.
- `PATTERN-markdown-link-integrity`: every `docs/` link to the renamed spec resolves.
- CLAUDE.md "Docs land in the same commit": `doc.go`, `docs/overview.md` and cobra help change with the code; a package pushed past its `cmd/lyx/testdata/test-budget.yaml` row lists that file under `Edit:`.

## Auto-mode assumptions

- A1: The operator's board text names `contracts/specs/final-summary-spec.md` as the darn stencil's spec path; D2 deploys and renames the spec instead, because the repo path exists only in loomyard's own repository and `PATTERN-stencil-ownership` routes specs through the deployed specs dir.
- A2: The landing describe stencil's bare `lyx board get {{.slug}}` is the same defect as darn's and is fixed in this task.
- A3: The reconcile fix covers floors, not "old template default" values: reconcile keeps no record of earlier templates, and every observed case is a below-floor value.
- A4: The confirmation flag is named `--apply`, matching fabric's existing destructive verbs, not a new `--yes` or `--confirm`.
- A5: `WriteCarryOver`'s pre-rename check compares the whole-file bytes on disk with the bytes it read at its start, rather than re-running the splice check on the disk copy; byte equality implies the splice holds.

## Open risks

- R1: An operator-edited copy of the darn stencil in a hub's deployed stencils dir is never overwritten (`stencilstore` leaves a hash-mismatched file), so that hub keeps the old prompt until the operator resolves the drift.
- R2: D8's checks hold no lock, so two windows stay open.
  A write landing between the pre-rename compare and the rename is overwritten by the rename with no error.
  A read-modify-write such as `lyx loom decision add`, which rewrites the record in place, that reads before the rename and writes after the post-rename re-read drops the carry-over entry with no error.
- R3: Renaming the spec changes `lyx stencil list`'s spec names and leaves any already-deployed `final-summary-spec.md` copy absent (none is deployed today).
- R4: The batten status lives in the durable run directory and the terminal-opened marker in the ephemeral scratch directory; a batten status that vanishes mid-run makes the next `lyx batten run` or `step` seed a fresh status and clear the marker.
A second window opens beside the still-attached first one only when InnerRun then spawns the child; a child it adopts without a spawn opens none.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d5-nonint-rule-loader-premise (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The rendered darn writer prompt contains `lyx board get '{"slug":"<slug>"}'` and `<specs_dir>/loom/loom-final-summary-spec.md`, and the rendered landing describe prompt contains the JSON `board get` form.
- `contracts/specs`'s registry lists `loom-plan-spec` and `loom-final-summary-spec`, both placed under `loom/`, and seeding a specs dir writes the final-summary spec there.
- No file in the repository references `final-summary-spec.md` by its old name.
- `docs/overview.md`'s batten paragraph names `loom`, `darn` or empty as the accepted recipes.
- InnerRun with the terminal-opened marker present and no child status file re-spawns the child without opening a second window; a fresh batten run (no batten status), started by `lyx batten run` or by `lyx batten step`, clears the marker so its first spawn opens one.
- `lyx config reconcile` on a worktree whose `shuttle.yaml` holds `poll_interval_ms` below the floor reports one `Migrated` line for it on a dry run without writing, and with `--apply` writes the same file reconcile writes for that input with the key already at the floor value; the same holds for `orch.poll_interval_ms` and `loom.step_idle_timeout_min`, and for a null value, reported with old value `null`; a value at or above the floor, an `${env:...}` marker, or any other value that does not decode as an integer yields no line and is left as it was.
- `lyx fabric unwire`, `lyx board prune` and `lyx board retire-legacy` without `--apply` exit 0, report `"applied": false` with what they would change, and leave junctions, exclude lines, board entries and legacy files unchanged; with `--apply` each behaves as today and reports `"applied": true`; `retire-legacy` reports `legacy_files` in both modes.
- `retire-legacy` with neither legacy file refuses with and without `--apply`.
- Every lyx-shipped caller of the three verbs passes `--apply`.
- The internal-audience command index carries no board payload legend; the operator-audience index carries it on the `board` line.
- A command carrying an index note without a valid note audience fails a test.
- `WriteCarryOver` refuses and leaves the record as the other writer left it when the bytes on disk before its rename differ from the bytes it read at its start.
- `WriteCarryOver` returns an error when the on-disk record after the rename differs from the bytes it wrote.
- The refusal-spec loom section has a row for the concurrent-write refusal, and a test reaches it.

## Notes for the plan writer

- `internal/configsync`'s `ReconcileAll` already calls `Module.Migrate` and reports `Migrated`; `batcher.MigrateConfig` is the existing hook to model the YAML rewrite on.
- `ReconcileHubWideAt` does not call `Migrate`; none of D5's modules is hub-wide.
- Floor constants: `shuttleengine`'s `pollFloor`, `orchengine`'s `pollFloor`, `loomengine`'s `stepIdleTimeoutFloorMin`.
- The InnerRun marker helpers live in `internal/battenshed/innerrun.go`; batten's fresh-status seeding is in `internal/battencli`'s pre-run and pre-step hooks.
- `internal/clihelp/index.go`'s `withNote` is the one place a note is appended.
- The refusal-spec rows "carry-over entry refused" and "carry-over write failed" cover the existing carry-over failures; D8's row sits beside them.
