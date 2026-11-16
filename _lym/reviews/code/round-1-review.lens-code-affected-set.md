```yaml
lens: code-affected-set
outcome: reported
```

Task: review the code-affected-set lens. For each change the diff makes, I looked for the callers, tests, docs, fixtures and rules it forces, and judged whether the diff leaves any of them wrong.

**Per change, the affected set and whether it is covered**

1. **`entryCell` output now adds `pull_request` / `on_land` bullets.**
   - Callers: `render.go` 636 (wait tables), 658 (Running), 692 (Notes) and 718 (Done non-project). The bullets appear in every one of these tables, which is consistent. Covered.
   - Tests: `TestRenderReadmeGolden` (`render_test.go` 374 and 382) is updated. No other test asserts entry-cell bullets; the grep for ``<br>• ` `` hits only `render_test.go`. Covered.
   - Fixture: `readmeFixture` is used only at `render_test.go:342`, so no other golden breaks. Covered.
   - Package doc entry-cell paragraph (`doc.go` 365–368) is updated. Covered.
   - Package doc profile paragraph (`doc.go:141`): "The README entry cell names only a profile the task carries." This is wrong. The profile renders in the Recipe cell through `recipeCell`, never in the entry cell. The diff's own design is that profile gets no entry-cell bullet, so this paragraph about entry-cell contents now contradicts the updated paragraph. **Not covered.**
   - `docs/overview.md` 469–480 describes the README sections, not the entry cell's contents. Not affected.
   - Stencils and specs: no hit in the grep. Not affected.
2. **New `runFieldLines`.** It is used only by `entryCell`. Covered.
3. **Commit messages** (`f5c405b9`, `f5b95ee5`, `7ffae958`): no `Pattern-Departure:` trailer, and the claims match `render.go`. Covered.
4. **Declared PATTERN changes:** the directive declares none. Nothing to land.
5. **Repository rules (`PATTERN-code-conventions`, named constants):** the board.json key names `pull_request` and `on_land` already repeat as literals (`replay.go:207`, `replay.go:214`, `front.go:33`, `front.go:57`, `store.go:500`), and `render.go` adds two more. **Not covered** (see F1).

**Findings**

- **F1 — BLOCKING, `design`.** `internal/boardengine/render.go:883` and `:886`. `runFieldLines` hard-codes the key literals `"pull_request: "` and `"on_land: "`. These repeat the board.json key names that the struct tags at `task.go:49` and `:67` define, and that `replay.go:207/214`, `front.go:33/57` and `store.go:500` already repeat. A rename of the key would leave the README line stale without any failure. `pattern/PATTERN-code-conventions.md:11` says "Writing a name or path that can go stale: one named constant, declared once and referenced everywhere, never a repeated literal." The diff adds new repeats of names that already exist as debt.
  - Fix: declare one named constant per key in `boardengine` and use it in `runFieldLines`.
  - For the handler: no such constant exists today, and the existing sites carry the same debt. Judge whether a JSON key counts as "a name that can go stale" under this rule.
- **F2 — LOW, `consistency`.** `internal/boardengine/doc.go:141` says "The README entry cell names only a profile the task carries." `recipeCell` (`render.go` around 955) puts the profile in the Recipe cell, `entryCell` never renders it, and the diff's own entry-cell paragraph (`doc.go` 365–368) and summary say profile gets no entry-cell bullet. The sentence predates the diff, but it describes the entry cell, which is the content this change redefines, so it now contradicts the doc paragraph the diff rewrote.
  - Fix: change it to say the README's Recipe cell names only a profile the task carries.

Outside this lens, in one sentence: the summary's claim that "owner_machine already shows in the Owner column" holds only for the Running table, because the Owner column is at `render.go:658` only.

**Tool calls**

1. A grep for tests, docs and stencils that assert or describe the entry cell. The manifest held no search outside `render.go` and `doc.go`.
2. Read `doc.go` 132–146 and 358–372, plus a grep for stencils and specs. This was to see the `doc.go:141` hit in context, which the manifest had not read.
3. A grep of `PATTERN-code-conventions.md` and of the other `"pull_request"`/`"on_land"` literals. The manifest had not read the background file or the other key sites.
4. Read `pattern/PATTERN-code-conventions.md` whole, to check the named-constants rule's exact scope.
