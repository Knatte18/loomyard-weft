# Card 21 — Commit the redundancy report and the post-retag timing

**Create:**
- `docs/benchmarks/test-redundancy.md`

**Prosa:**
- `docs/benchmarks/test-suite-timing.md`
- `docs/benchmarks/running-tests.md`

**Uses:**
- `cmd/testtiming/redundancy.go`

**Intent:**
Generate the redundancy report with `go run ./cmd/testtiming -redundancy` on this tree (every tier but `llm`, after every earlier card) and commit `docs/benchmarks/test-redundancy.md` exactly as the tool writes it (D3).
The generated sections are never hand-edited.
Check acceptance on the result: every package has its count, wall and serial time; candidates name their covering tests; the removable set is marked; and every test that spawns a subprocess (for example the `lyxbin`-driven `cmd/lyx` tests and the reedcli `tmux` tests) is listed under "no coverage", never as a candidate.
If one is not, the scan misses a shape: report it as a defect of card 20 and fix it there before generating the report, since this card only runs the tool and writes docs.

In `docs/benchmarks/test-suite-timing.md`'s dated section, add the post-retag numbers with card 2's method: Tier 1, `-tags integration` and `-tags tmux` wall times, each tier's test count, revision and load (D4).
State that the integration tier's test set changed at the retag, so these numbers do not compare with the before and after-speed ones.
Link the new redundancy report from this file and from `docs/benchmarks/running-tests.md`'s timing-harness section, with links that resolve (`PATTERN-markdown-link-integrity`).
The PR description quotes the before, after-speed and post-retag numbers from this section.
