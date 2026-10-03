# Card 5 — round-facts-file

**Edit:**
- `internal/shedadapters/round.go`

**Create:**
- `internal/shedadapters/bouncerfacts.go`
- `internal/shedadapters/bouncerfacts_test.go`

**Uses:**
- `internal/burlerengine/verdict.go`
- `internal/shedadapters/bouncerfiles.go`

**Intent:** Compute and render the per-round facts file the judge reads instead of the artifacts (D3).

`round.go` gains `factsPath(runDir, round)`, naming `round-<N>-facts.md` beside the verdict and ledger paths.
It is not a judge output, so `judgeOutputs` is unchanged.

`bouncerfacts.go` holds three pieces, so the computation is testable apart from the file write:

- `computeRoundFacts(runDir string, round int, reportName func(int) string) roundFacts` reads rounds 1..N's review files through `burlerengine.ParseReview`, never a second parser.
  Per round it records the finding count, the count per severity, the highest severity, the count per class, and the gating count: findings whose class is `burlerengine.GatingClass` at MEDIUM or BLOCKING severity.
  A review that is missing or fails to parse yields a row carrying the error text instead of counts, and never an error from the function.
  The review's own top-level `verdict:` is read by the parser but never recorded.
  From the ledgers of rounds 1..N-1 (through `parseLedger`, skipping a ledger that fails to parse) it collects every recurring key: a key whose `rounds` hold two or more distinct rounds, with those rounds, marked reopened when an earlier ledger has it `resolved` and a later ledger has it `open`.
- `renderRoundFacts(f roundFacts) []byte` writes deterministic Markdown: a heading naming round N;
  one table with a row per round and columns for findings, each severity, highest severity, each class and gating count, so rise, fall or plateau reads down a column;
  a parse-error row spanning its counts with the error text;
  then a recurring-keys list, or a line saying there are none.
  Classes and severities appear in their constant order, keys sorted, so two renders of the same run directory are byte-identical.
- `writeRoundFacts(runDir string, round int, reportName func(int) string) error` computes, renders and writes the file at `factsPath`, overwriting any earlier render.

Tests in `bouncerfacts_test.go`, over a temp run directory:
a fixture whose rounds 1 to 3 are judged renders `round-4-facts.md` with per-round finding counts, per-severity counts, highest severity, per-class counts and gating counts matching the fixtures, and lists a key as recurring and reopened when round 2's ledger has it `resolved` and round 3's ledger has it `open` with rounds `[1, 3]`;
a key seen in one round only is not listed;
a round-1 review with no `class` renders a parse-error row for round 1 while round 2's counts still render, and `writeRoundFacts` returns nil;
the rework-generations fixture, a round-1 review with MEDIUM `design` findings beside a fixer report claiming all fixed, renders a non-zero gating count for round 1;
a BLOCKING `scope` finding counts in the BLOCKING column but not the gating count;
rendering twice yields identical bytes.

**ImpactSummary:** Adds one path helper beside the existing ones in `round.go`; nothing calls the facts writer until card 6.

**Commit:** 5: bouncer: render a per-round facts file from the reviews and ledgers
