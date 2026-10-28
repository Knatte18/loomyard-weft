## Goal

Run the first recurring test-suite time and growth audit and append its numbers as a new section of `docs/benchmarks/test-suite-timing.md`.
The section lets a later audit compare wall time per tier and per-package test counts against this one.

## Scope

- In: one new measurement-report section in `docs/benchmarks/test-suite-timing.md`, after the last section, under the file's stated method.
- In: wall time for `tier0`, `tier1` and `tier2`, each the median of exactly 3 warm runs after `go build ./...`.
- In: wall time per tier compared with the latest after-state in the file (`test-tiers`, 2026-10-07), the packages that moved most, and the per-package `TESTS` growth since then.
- In: a mapping from the new tier tags to the file's older Tier 1 and Tier 2 columns, and the prune-task candidates as lines in the section.
- Out: `tier3` (tmux) and every LLM tier.
- Out: any change to the older rows, to `cmd/testtiming` or to any other file; no board task is filed.

## Decisions

### D1 — Section placement and form

The section is appended at the end of the file as an H2 `## test-time-audit: <date> audit`, with H3 subsections and a fenced `yaml` metadata block, as the existing measurement-report sections are.
Rationale: the file's convention is one self-contained report per measurement, and the board entry asks for a new section; the older rows stay as written.

### D2 — Measurement method

Per tier, run `go build ./...` once, then four `go run ./cmd/testtiming -tags <tier>` runs, discard the first as cold, and report the median wall of the other three.
Rationale: the file's method names the median of 3 warm runs, and the prior sections discard a cold first run because `go build` does not link test binaries; the board entry says "exactly 3" warm runs.

### D3 — Tier naming and mapping

The section names the tiers by tag and states that `tier0` is the older Tier 1 offline loop, and `tier1` and `tier2` together are the git-spawning part of the older Tier 2 and integration loops, without claiming the sets are equal.
Rationale: the tags are disjoint and do not nest, so a tag's wall is not comparable one to one with an older column; the section says which comparisons hold and which do not.

### D4 — Quiet-machine precondition

Before measuring, run `lym tender status` and `lym gate status` and record load average and what else runs; a busy machine is recorded in the section, not hidden or waited out.
Rationale: the entry requires the check and the older sections record load the same way; the recorded condition is the reader's means to discount a number.

### D5 — Refused run route

If lym refuses a `go run ./cmd/testtiming` call under the agent's run rules, take the route the refusal names and record the route in the section.
Bound: the route is whatever the refusal names; no gate or slot rule is bypassed, and nothing is run that the refusal does not name.
Rationale: the entry says so, and the measured number is only meaningful with its route stated.

### D6 — Growth and prune candidates

Per-package growth is the difference between this audit's `TESTS` column and the previous recorded after-state's counts; where the file holds only per-tier totals for the previous state, the section compares totals and says the per-package baseline is absent.
A package that grew with no matching new behavior is listed as one line, with its counts, as a candidate for a prune task under `PATTERN-test-speed`; the writer files no task.
Rationale: the entry asks for exactly this and forbids filing; reading a package's new behavior is a coarse judgment, so each candidate says what was checked.

### D7 — Commit

The change is one docs-only commit on the task branch.
Rationale: `CLAUDE.md` exempts `docs/` text no build or test reads from the test run, and no module, CLI behavior or PATTERN entry changes.

## Constraints

- `PATTERN-test-speed`: test runs go through lym's gate; the measurement uses `cmd/testtiming`, which holds one gate slot per run, and the writer runs no raw module-wide test.
- `PATTERN-gate-slot`: `cmd/testtiming` holds the gate slot itself; the writer takes no slot by hand.
- `CLAUDE.md`: semantic line breaks in the markdown; no production install; no push.

## Auto-mode assumptions

- The audit date is 2026-10-28, the board entry's run date.
- The latest recorded after-state is the `test-tiers` After state (2026-10-07); `tier_1`, `integration` and `tmux` totals there are the only counts the file holds.

## Open risks

- The machine is shared with other runs in flight (`lym tender status` lists several batten runs), so the walls may be inflated; the section records the load and the processes seen.
- The old tier totals were taken before the gate's tier tags, so a tier-by-tier delta is indicative only.
- `go run ./cmd/testtiming` may be refused under the agent's run rules; D5 covers the route.

## Acceptance criteria

- D1: the file ends with one new H2 section holding a `yaml` block with machine, os, go, revision, load averages, per-tier walls and per-tier top-level test counts; no older line changes.
- D2: the section lists the three warm run walls of each of `tier0`, `tier1` and `tier2` and the median taken.
- D3: the section maps each tag to the older Tier 1 and Tier 2 names and states which comparisons are not one to one.
- D4: the section records the `lym tender status` and `lym gate status` findings and the load average before and after.
- D5: any refused call and the route taken are recorded; with no refusal the section says the direct call ran.
- D6: the section names the per-package growth, or states that the baseline is per-tier only, and lists the prune candidates as lines; no board task is created.
- D7: one docs-only commit; `lym loom validate-description` is clean.
