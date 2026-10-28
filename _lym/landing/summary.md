# Tune review lenses and run a solo code review in light, docs, speed

Builds the lens and code-review recommendations of `docs/research/review-setup-audit-2026-10-28/report.md`, which the operator accepted on 2026-10-28.
The audit found some lenses noisy with no gating finds, and the code fan's first rounds mostly empty.

## Lens changes (`internal/lenses/template.yaml`, `sets/csharp.yaml`)

- `board-scope` replaces `goal-scope` and `board-fit`.
  It reads the board, `lym loom peers` and a few nearby decision notes (at most six tool calls) and reports only contradictions with the board.
  Doubt about what the task is for goes out as a `decision`-class question for the run's parent.
  It sits in `twill-discussion` and the dormant `discussion` fan; `review-discussion` drops the pair and gets no replacement.
- `repo-invariants` replaces `way-forward` in `review-plan`.
  It checks the invariants a card breaks without touching the line that breaks: test seams and tier tags, `doc.go` and PATTERN text a card falsifies, and card order.
  `way-forward` stays defined and stays in `twill-plan`, and its prose now ends with the emphasis line.
- `code-pattern-fit` is deleted and folded into `code-affected-set`, whose set now includes the repository's rule documents (`PATTERN.md`, `pattern/`, or the `CLAUDE.md` rules).
  The `csharp` layer's `review-code` drops it too.
- `code-tests` is narrowed to missing coverage of changed behaviour and no longer applies "Test economy"; the review lead still judges that from the unchanged rubric.

## Solo code review in light, docs and speed

- `loom.code_reviewer_fan` is reclassified from `CategoryGuard` to `CategoryNone`, so a profile block, `run_settings` and a pair's local layer can set it.
  The other five reviewer and fixer fan keys stay guards.
  The retired keys that move onto it (`darn_reviewer_fan`, `review_fan`, `webster_reviewer_fan`, `webster_fan`) are admitted with it.
- `internal/runprofile/template.yaml` sets `code_reviewer_fan: ""` in `light`'s `loom`, `darn` and `pawl` blocks, `speed`'s `loom` block and `docs`'s `darn` block.
  `solid` and `wide` keep the hub's fan; `speed`'s `darn` and `pawl` blocks run no code round.

## Design decisions to know

- The fan name is not checked against `lenses.yaml` at write time; a wrong name fails the pair's loom config load through `ValidateFanKeys`, which names the key.
- An agent can now thin its own run's code review to solo or point it at another fan through its local layer.
  The reviewer's whole-diff read under the code-review rubric stays the floor, and `code_max_bounces: 0` remains the stronger switch it already had.
- Deleted lenses go outright, with no alias.
- `TestConfigTemplate` widens its emphasis-line exemption to `board-scope`.
  The writer-covers-reviewer check gets `repo-invariants` as its one reviewer-only lens.
- Docs updated: `internal/runprofile/doc.go`, `docs/overview.md`, `docs/dotnet-hub.md`, `pattern/PATTERN-config-layers.md` and the `internal/loomengine/config.go` comments.

## Check by hand after landing

- `profiles.yaml` is seed-only, so the hub's live `_board/_lym/config/profiles.yaml` does not get the new blocks.
  The operator re-sets `light`, `docs` and `speed`'s loom block with `lym config profiles --set`.
  The live `docs` block has `code_max_bounces: 0`, so the fan setting is moot there until that changes.
- A hub whose `lenses.yaml` has a fan naming `goal-scope`, `board-fit` or `code-pattern-fit` fails every loom config load once this deploys, when a reviewer or fixer fan key names that fan.
  `docs/dotnet-hub.md` shows the OSL-1046 Models hub with such a `review-code` fan; drop the names with `lym config lenses --set` before the deploy.
