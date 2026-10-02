# The board becomes the roadmap

All ten batches reported `status: done`, in list order: 01, 02, 03, 04, 06, 05, 07, 08, 09, 10.
Each ran as one fork and none needed recovery.

- **01 entry-schema** added `tier` and `type` to the entry, renamed the recipe field, retired `deferred`, and enforced the tier dependency rule.
- **02 store-ops** added `Promote`, `Prune` and `Find` on the in-memory store.
- **03 legacy-migration** added `legacy.go`, the pure migration and done-mark fold from the legacy files.
- **04 board-json-store** switched the board to one `board.json` store, migrating on load, and collapsed the two-store facade.
- **06 render-roadmap** renders the README per tier and gives design docs a header.
- **05 board-verbs** exposed `Promote`, `Prune`, `Find` and `RetireLegacy` on the `Board` facade.
- **07 cli-verbs** added `promote`, `prune`, `find`, `retire-legacy` and `--text`, and made `notes` and `promote-note` hidden aliases.
- **08 cli-help** documented every payload verb's keys in its `--help`.
- **09 board-skill** added the `ly:board` skill and listed it in the ly index.
- **10 overview-doc** updated the board entry in `docs/overview.md`.

Reported deviation: batch 01 also changed `internal/boardengine/boardtest/bench_test.go`, outside its declared file-ops, because its seeded tasks had tier 0 and failed the new validation.

Batch 10's fork noted that the batten bullet in `docs/overview.md` still says a board task's `type` must be `loom` or empty, which describes the old recipe field; the card did not cover it.
Batch 07's fork noted that `upsert`'s help still named the retired `deferred` field; batch 08 fixed it.

The integration fork ran the plan-level `## verify:` command once at HEAD `4594739bc`, and its report says `status: OK`.
