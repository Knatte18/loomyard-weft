## Goal

```markdown
## Seen (orch, 2026-11-08)

The first cut (`f2ac2bf7`) dropped 6 `internal/fabricengine` tests and added 8 tier 0 tests: 760 tests before, 762 after, and no time measured.
Its body told the writer to confirm each drop against the module-wide `go run ./cmd/testtiming -redundancy` check,
which counts the fixture packages' blocks too and lists almost no fabricengine test as contained, so it vetoed nearly every drop.
That check was the fault, not the writer.

## Change (operator, 2026-11-08)

Drop the tests in the coverage study's removable set: `greedy.tsv` in `docs/research/speed/fabric-test-redundancy-2026-11-08/`,
the tests that can go together without leaving a package-scoped block uncovered, slowest first (its cost column).
Skip the tests the first cut already dropped or folded, and those already carrying `//testtiming:keep`.

- The study's list is the authority. No other check vetoes a drop; do not run or consult the module-wide `-redundancy` check.
- The default is to drop. Read each test before it goes, and keep it only when it asserts something concrete block coverage cannot see:
  archive tags and their order, deletes on origin, user content never claimed or removed, mutation records and their order,
  machine-local files kept out of the weft tree.
  A kept test carries `//testtiming:keep <reason>` naming what it pins.
- Drop only tests in the set, which keeps every rarest-block partner; never hand-pick a test outside it.
- Production code does not change.

## How the writer works

The work is too large for the darn writer alone: it delegates to subagents through the Agent tool.
Split the tests into batches by source file, one subagent per batch, never two subagents on the same file.
Each reads its tests, drops or keeps them under the rules above, and makes the edits.
The writer reads their reports, settles anything they flag and runs the gate.

## Numbers in the PR

The task is not done without them:

- the count of tests dropped and the count kept, with each keep's reason listed;
- the tier 1 time of `internal/fabricengine` in one gate pass before the first edit and one after the last, on the same machine.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
