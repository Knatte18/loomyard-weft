# Card 17 — measure-after

**Edit:**
- `docs/benchmarks/test-suite-prune-core.md`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
D8's after half, and the final check of the acceptance criteria.

Rerun `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg ./internal/fabricengine,./internal/websterengine,./internal/reedengine,./internal/shuttleengine,./internal/loomcli,./internal/shedadapters` and commit the six sections.
Each section lists no candidate outside "Kept", no test as "failed", and every "Kept" entry has a reason; when one does not, fix it in the package's test files in this card, and note it in the commit message.

Repeat card 5's measurements on the same machine with the same commands: the three timing runs (medians), the coverage run, and the load average of each.
Fill the after section of `docs/benchmarks/test-suite-prune-core.md` with the same table shape as the before section, plus the delta per package.
A package whose wall time rose names, below the table, the scenario merges behind it (cards 9, 11, 13 and 15 name each merged family in their commit messages).
Compare each package's coverage with the before figure; when it dropped, compare the before and after coverage profiles (card 5 kept the before ones under `.scratch/`, or regenerate them from the measured commit), and list each lost block with the deleted test that alone covered it.

The after numbers go only in this report; `docs/benchmarks/test-suite-timing.md` stays untouched (`test-tiers` owns it).

**ImpactSummary:** Fills the after half of the measurement report and rewrites the six report sections, plus test-file fixes in the six packages only if the final rerun finds a gap.
