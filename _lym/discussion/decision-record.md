# Decision record — claude-md-lym-rules-out

## Goal

Delete the repository-root `CLAUDE.md` once every rule that stands only in it has a home that reaches the agents needing it, as the operator decided from `docs/research/claude-md-rule-map-2026-10-29.md`.

## Scope

- Move the PATTERN-in-the-same-commit rule and the docs-in-the-same-commit list into `contracts/stencils/darn/darn-template-write.md` step 4.3, and drop its step-1 pointer to `CLAUDE.md`.
- Replace the `CLAUDE.md` pointer in `contracts/stencils/drift/drift-template-baseline.md` step 1.
- Write the production rule ("never run `update-plugins.sh` unasked, never install `lym` elsewhere") and the hand-push gate (`--upto 2`, own Bash call, docs-only exemption) into the hub-wide orch rules file through `lym orch rules`, from a copy of the current file.
- Rewrite the placement pointer in `contracts/stencils/orch/orch-template-role.md` (Where a finding goes) to name the orch rules file and the stencils.
- Add the own-worktree rule to `contracts/stencils/edit/edit-directive.md` and its package doc in `internal/editdirective/doc.go`.
- Delete `CLAUDE.md`; rewrite the comment on `merriamFixedContext` in `internal/websterengine/merriambase.go`.
- Fix the remaining comments and docs that cite this repository's `CLAUDE.md` as a live rule.
- Out of scope: the rows the report marks `covered`, the engine's instruction-file list in `internal/shuttleengine/claudeengine/capabilities.go`, and historical or plan docs under `docs/research/`, `docs/skills.md` and `crucible/`.

## Decisions

### D1 — Darn step 4.3 states the docs rule generically

Step 4.3 names the docs a change carries in its commits: the package doc of each package whose behavior changes (`doc.go` in Go), the repository overview when a module table or execution stack it shows changes, and a `PATTERN.md` entry for a new cross-cutting invariant when the repository has `PATTERN.md`.
Step 1's "read the repository's `CLAUDE.md`" line goes.
Rationale: the darn stencil ships to every hub, so the rule is phrased to hold in a repository without `docs/overview.md` or `PATTERN.md`; Claude Code still loads a repository's own instruction file on its own.

### D2 — Drift baseline gets a pointer replacement only

Step 1's `CLAUDE.md` line becomes a read of the repository's README and its test setup for how it is built and tested.
The docs and PATTERN rules do not go into the baseline stencil.
Rationale: the baseline agent changes no code, so a rule about docs landing with code has nothing to act on there; the report's claim that this line names the doc list misreads it, since it names the build and test route.

### D3 — The orch rules file states the production and hand-push rules itself

The rules file's Production section, which today is an exception to "`CLAUDE.md`'s Production rule", is rewritten to state the rule and keep the exception: `update-plugins.sh` is the only route to production, never run unasked and never install `lym` elsewhere, except after the orch's own pushed fix on a clean `main`.
A new section states the hand-push gate: push `main` only from the worktree on `main`, only after `lym gate test --upto 2` is green there as its own Bash call, and a commit changing only text no build or test reads is pushed without the run.
The draft starts from a copy of the current file under `.scratch/` and is written with `lym orch rules --from`.
Bound: the verb replaces the whole hub file, so the draft keeps every other section byte for byte.

### D4 — The orch role's placement rule names the rules file and the stencils

The Where-a-finding-goes sentence names `lym orch rules` for a workaround only orchs apply and the stencils lym ships for one spawned agents need, in place of `CLAUDE.md`.
Rationale: the file it named goes, and LoomYard adds no instruction file to a repository.

### D5 — The edit directive carries the own-worktree rule

The directive adds: work only in your own worktree, never edit, commit or push in another and never create one unless your parent or the operator says so for that case; when work belongs elsewhere, say so to your parent.
`internal/editdirective/doc.go` and the stencil's header comment follow.
Rationale: the directive is rendered into every spawned role, so the rule reaches each of them; a spawned role has a parent rather than an operator in its pane.

### D6 — `merriamFixedContext` keeps its value

The constant stays `20600 - 392*12`; only its comment changes, to say that the measured start included a 59-line instruction file and that the fixed figure excludes every text the base counts, so it holds for a tree without that file.
Rationale: the fixed figure is the system prompt, tools and skills, which deleting `CLAUDE.md` does not change; the base drops the file's lines by itself, since an absent instruction file adds nothing.

### D7 — Stale pointers to the deleted file are fixed

The comments in `cmd/lym/crosscompile_test.go`, `tools/mdreflow/reflow.go` and `internal/fabricengine/doc.go` stop citing this repository's `CLAUDE.md` and name README, `scribe:prose` and `PATTERN-fs-links` instead.
`CLAUDE.md` leaves the vocabulary scan's root-file set in `internal/lymcwd/enforcement_test.go` and the scope list in `pattern/PATTERN-fabric-vocabulary.md`.
Rationale: each would otherwise name a file that no longer exists as the holder of a rule.

### D8 — The background line and the shorthand rule are dropped

The opening line about LoomYard developing itself and the conversational-shorthand rule get no new home, as the board entry says.

### Added after Discussion (parent, 2026-10-29): D2 stands: the drift baseline stencil gets the pointer replacement only

Decision: Decision D2 of the record stands and the board entry's scope is corrected to it: in contracts/stencils/drift/drift-template-baseline.md the CLAUDE.md line in step 1 is replaced by a read of the repository's README and test setup, and the docs-in-the-same-commit and PATTERN rule is not added to the baseline stencil. Code-Review finding F1 is closed without a change to the stencil.

Rationale: The baseline agent changes no code, so a rule about docs landing with code has nothing to act on there, and the line in step 1 names the build and test route, which the rule-map report read as the doc list.

## Constraints

- `PATTERN-stencil-ownership`: stencils are read at call time; the edits change stencil text only.
- `PATTERN-fabric-vocabulary`: the scan's scope list and its background file change together.
- Markdown follows semantic line breaks.
- The task does not edit its own board entry.

## Auto-mode assumptions

- Writing the hub orch rules file through `lym orch rules` from this run is allowed, since the board entry names that verb as the route.
- The interactive-tmux and filesystem-links rules already landed as `PATTERN-interactive-spawn` and `PATTERN-fs-links`, so this task moves neither.

## Open risks

- `lym orch rules` may refuse a task session; then the rules-file draft goes to the parent to write.
- The orch rules file is written on Bolt before this run lands; its new text names no file this task deletes, so it stands either way.

## Acceptance criteria

- D1: the darn write stencil names no `CLAUDE.md` and its step 4.3 states the docs and PATTERN rule.
- D2: the drift baseline stencil names no `CLAUDE.md`.
- D3: the hub orch rules file states the production rule with its exception and the hand-push gate, and every other section is unchanged.
- D4: the orch role stencil names no `CLAUDE.md`.
- D5: the edit directive states the own-worktree rule and its package doc says so.
- D6: `merriamFixedContext` keeps its value and its comment names no `CLAUDE.md`; the merriambase tests pass.
- D7: no comment, scan list or PATTERN background file cites the repository's `CLAUDE.md`; the vocabulary scan passes.
- D8: `CLAUDE.md` is deleted and the gate run over the change is green.
