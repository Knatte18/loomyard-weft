# Card 3 — board-scope-lens

**Edit:**
- `internal/lenses/template.yaml`
- `internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent`
- `internal/lenses#TestLoad`
- `internal/loomengine#TestDiscussionTable`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`

**Create:**
- `plan:internal/lenses#emphasisLineGaps` -> `func emphasisLineGaps(lensTexts map[string]string, exempt map[string]bool) []string`
- `plan:internal/lenses#TestTemplateChecks_FlagBrokenTemplates` -> `func TestTemplateChecks_FlagBrokenTemplates(t *testing.T)`

**Uses:**
- `internal/lenses#loadTemplateConfig`
- `internal/lenses#ResolveFan`

**Intent:**
D1: in `internal/lenses/template.yaml`, add the procedure lens `board-scope` and delete `goal-scope` and `board-fit`.
`twill-discussion` becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `board-scope`, `leanness`;
`review-discussion` becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`;
the dormant `discussion` fan becomes `generic`, `board-scope`, `pattern-fit`, `scenarios`, `leanness`.

`board-scope`'s prose, one paragraph in the style of the other procedure lenses, says each of D1's points:
- It reads `lym board list`, `lym board get` for this task and `lym loom peers`, then up to three reads, nearest first, of a `Decision for <slug>:` note for this task, a touching entry, a close peer's record or a parent commit `lym loom peers` lists, and says "that is at most six tool calls".
- A `Decision for <slug>:` note for this task and the record's `Added after Discussion` entries are parent decisions and win over the entry body.
- It reports only a contradiction with the board: a goal point of the entry the record leaves out or defers, work the entry rules out, work another entry owns, and a design conflict with another entry, run or landed parent commit.
- Doubt about what the task is for, an entry plainly outdated by landed work and a conflict the writer cannot settle are a finding of class `decision`, phrased as a question for the run's parent; everything else is an ordinary finding.
- A text-only overlap, a shared bookkeeping line and a question a decision note or `Added after Discussion` entry settles are no finding, said without the word "ignore".
- It ends in its own list, opening "Your list:": each goal point met or not, each entry, peer and commit checked with conflict or none, and each decision note read or unreachable; no emphasis line follows.

The template header's procedure-lens sentence widens to name `board-scope`: the procedure lenses are those of the production fans (each `review-*` and `fix-*` fan) and `board-scope`, each a procedure ending in its own list.

Tests in `internal/lenses`:
- `emphasisLineGaps` is a test helper in `config_test.go`: it returns, sorted, the names in `lensTexts`, other than `generic` and those `exempt` holds, whose whitespace-normalised text does not end with the emphasis line.
  The emphasis line becomes a package-level test const, `emphasisLine`, which `TestConfigTemplate_DecodesAndIsSelfConsistent` and the helper share.
- `TestConfigTemplate_DecodesAndIsSelfConsistent`: `wantLenses` becomes 37; `twill-discussion` 6 entries and `review-discussion` 4; `wantOrders` carries the three new fan orders.
  After the procedure set is computed from the `review-*` and `fix-*` orders, `procedureLenses["board-scope"] = true`, and the base-fan set used for layer lenses gains `board-scope` the same way.
  The base and per-layer emphasis checks call `emphasisLineGaps` and report one error per name it returns.
  `wantSentences` gains `board-scope`: "that is at most six tool calls", "reports only a contradiction with the board", "class `decision`" and "Your list:".
  The test also asserts that `board-scope`'s text does not contain the emphasis line, so nothing follows its list, that `cfg.Lenses` defines neither `goal-scope` nor `board-fit`, and that the computed `procedureLenses` holds `board-scope` and lacks `leanness`, so the exempt set the template check builds admits only the lens D1 names.
- `TestTemplateChecks_FlagBrokenTemplates` table-drives `emphasisLineGaps` over hand-built lens maps with `exempt` holding `board-scope`: `board-scope` ending in its list gives none; `leanness` and `scenarios` ending without the emphasis line give `[leanness scenarios]`, in sorted order; `generic` without it gives none; a lens ending with the line gives none.
  This pins D1's criterion that the exemption admits `board-scope` and still fails any other lens of a non-procedure fan, `leanness` among them.
- `TestLoad`'s row "a hub lens keeps its own text over the template's while a sibling lens of the fan resolves the template's" tunes `interactions` instead of `goal-scope` and checks `existing-paths` resolves the template's text instead of `board-fit`.

Test in `internal/loomengine`:
- `TestDiscussionTable`'s "lens review" rows that redefine a fan with `[goal-scope]` use `[board-scope]` and want `board-scope`, and the dropped-lens check skips `board-scope` instead of `goal-scope`, since `goal-scope` no longer resolves.

Test in `internal/configsync`:
- `TestReconcile_MovesKeysAcrossModules`' fixture `burlerOldTemplate` cuts the template's `twill-discussion` fan to `[:5]` instead of `[:6]`, since the fan now has six lenses and `[:6]` would equal the template, leaving the row "a lenses.yaml equal to the template is kept over an old-template burler.yaml and cut to empty maps" without the old-template input it names.

A hub whose `lenses.yaml` names `goal-scope` or `board-fit` in a fan fails at resolve after the deploy; that is the record's open risk, for the operator, not a card's.
The refusal it meets, `ValidateFanKeys` over `ResolveFan`'s undefined-lens error, says to define the fan or set the entry to `""`; the operator's recovery is to drop the deleted lens name from the fan with `lym config lenses --set` or the editor, as the record's open risk says, and the shared refusal text stays as it is.

**ImpactSummary:** one template file and four tests in three packages; no Go production code reads a lens name, and no stencil names the deleted lenses.
