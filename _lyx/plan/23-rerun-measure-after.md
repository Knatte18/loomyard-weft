# Card 23 — rerun-measure-after

**Edit:**
- `docs/benchmarks/test-redundancy.md`
- `docs/benchmarks/test-suite-prune-rest.md`

**Uses:**
- `cmd/testtiming#`

**Intent:**
The final rerun and splice (Decision 9), the acceptance check of Decision 10, and the after half of Decisions 11 and 12.

Rerun `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg <package> -out <scratch path>` once per in-scope package, each to its own file under `.scratch/`.
`cmd/testtiming` is the one exception for the rerun's keep check, the gap fix and the splice: its tests are core's (the parent's `cmd/testtiming` addition to the decision record), so this task neither annotates nor edits them and leaves its report section as it is.
Its row in the before and after tables is still measured, read-only.
Compare each run's candidate list with the source: every listed candidate carries a non-empty `//testtiming:keep <reason>` directly above its `func Test…` line whose reason is one of the classes of the `prune-procedure`'s delete-or-keep step (the four of Decision 10, and the skill-loading reason `ownership-boundary` names, only on a test it lists), and no in-scope test is listed as `failed` unless its Decision 8 verdict is a tool artifact.
A gap is fixed in that package's test files in this card, under the prune procedure, and named in the commit message.
Then splice: each scratch file's `## <package>` section replaces the section of the same name in `docs/benchmarks/test-redundancy.md`, one section at a time, adding none; the header, the core packages' sections and the `cmd/testtiming` section stay byte-identical, which `git diff` shows.
Every link in the edited report and in `docs/benchmarks/test-suite-prune-rest.md` still resolves (`PATTERN-markdown-link-integrity`).

Repeat card 1's measurements with the same commands on the same machine: three timing runs (medians), the coverage run, and each run's load average.
Fill the after section of `docs/benchmarks/test-suite-prune-rest.md` with the same table shape as the before section plus the delta per package.
A package whose wall time rose names, below the table, the scenario merges behind it (each package card's commit message names its merged families).
A package whose coverage dropped is compared block by block with card 1's before profile (or one regenerated from the measured commit), and the lost block is restored by bringing back or extending a test in that package; only a block that also flips between two runs of the base commit is recorded in the report as noise, with that second run named (Decision 12).

`docs/benchmarks/test-suite-timing.md` stays untouched (`test-tiers` owns it).

**ImpactSummary:** Regenerates the in-scope sections of a generated report and fills the after half of this task's measurement report, plus in-scope test-file fixes only if the final rerun or coverage check finds a gap.

**Commit:** 23: Rerun the redundancy report and record the after measurement
