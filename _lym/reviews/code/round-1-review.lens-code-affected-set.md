```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether the diff left it wrong:**

- **Darn step 1 and 4.3** (`contracts/stencils/darn/darn-template-write.md`): no reference to this repo's `CLAUDE.md` remains in `contracts/`, `internal/` (non-test), `tools/` or `plugins/`. The only residual hit is the `internal/lenses/template.yaml` wording "its CLAUDE.md files", which is a generic phrase for a target repo. Covered.
- **Drift baseline step 1**: no residual reference. Covered.
- **Edit directive text** (`contracts/stencils/edit/edit-directive.md`, `internal/editdirective/doc.go`):
  - The doc comments that name `internal/editdirective` as renderer need no change, since they describe rendering and not the rule.
  - `pattern/PATTERN-parent-directive.md:13` was not updated (F2).
  - Not covered: no stencil I found contradicts the new rule, though I did not read every role that creates or edits a worktree.
- **Orch placement rule** (`orch-template-role.md:186-188`): the new text itself names `CLAUDE.md` (F1).
- **`merriamFixedContext` comment**: `internal/websterengine/merriambase_test.go` and the `webstercli` and `runlevel` tests only plant their own temporary `CLAUDE.md` for the engine's instruction-file behavior, which the engine still serves for other repos. No tool or test reads this repo's real `CLAUDE.md`. Covered.
- **Vocabulary scan** (`vocabScanRootFiles` and its comments in `internal/lymcwd/enforcement_test.go`, plus `pattern/PATTERN-fabric-vocabulary.md`): both sides lose `CLAUDE.md` together, and the scan doc comments are updated. Covered.
- **Stale-pointer comments** (`cmd/lym/crosscompile_test.go`, `tools/mdreflow/reflow.go`, `internal/fabricengine/doc.go`): `README.md:220` states the cgo requirement, so the rewritten pointer is true. Covered.
- **Declared PATTERN changes**: the directive declares none beyond `PATTERN-fabric-vocabulary`, which landed. Commit message and trailers: there is no `Pattern-Departure` trailer and none is needed.

**Findings**

- **F1 — LOW — consistency** — `contracts/stencils/orch/orch-template-role.md:188`
  The new sentence "lym adds no instruction file such as `CLAUDE.md` to a repository" names `CLAUDE.md` in the orch stencil. The decision record's D4 acceptance criterion is "the orch role stencil names no `CLAUDE.md`", so the diff misses it. Fix: drop the `CLAUDE.md` clause or reword it to "lym adds no instruction file to a repository".

- **F2 — NIT — consistency** — `pattern/PATTERN-parent-directive.md:13`
  It says the `edit-directive` stencil is "the only statement of the edit rule". The stencil now carries a second rule, own-worktree. The stencil header was changed to "its rules' only statement", but this background file was not. Fix: reword to "the only statement of the edit directive's rules".

**Tool calls (2)**
1. Grepped `CLAUDE.md` across `tools`, `plugins`, `contracts`, `cmd`, `pattern`, README, docs, scripts and non-test `internal`, because the earlier grep was cut off at 80 lines before reaching them.
2. Grepped for docs and PATTERN text describing the edit directive and the worktree rules, because my context did not hold other statements of them.
