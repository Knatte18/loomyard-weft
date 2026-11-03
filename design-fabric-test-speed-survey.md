# Map where fabricengine test time goes and propose cuts

`fabric-test-speed-survey` · Task · research, fabric, testkit, docs · running · Finalize

A report that maps where `internal/fabricengine`'s test time goes and proposes how to cut it, on both machines (operator, 2026-11-03).

## Why

`internal/fabricengine` is most of the suite's test time.
`docs/benchmarks/test-suite-timing.md` (`test-time-audit-3`) has it at 348.72 s of the 388.6 s `tier1` wall on OSL-1046, and at 45.69 s on neve in `test-time-audit-2`; the operator calls both too slow.
On OSL-1046, Cortex XDR scans every file opened and every process started, so the thousands of files and git processes the tests make each cost a scan.
Putting the temp root in memory (`fabricengine-shm-bench` in the same doc, and `0197761c`) did not help much.
The operator accepts cutting tests if that is what it takes.

## What the report answers

- Where the time goes on neve, per test file and per fixture builder, from timed runs through `lym gate test`.
- How many git processes and files a run makes, split into fixture setup and the code under test.
- What the existing template-and-copy model in `internal/hubforge` (`doc.go`: bares built once per binary through `sync.Once`, `CopyHub` and `SharedHub` per shape) already covers, and which fabricengine fixtures (`newMergePairFixture`, `newOriginHub`, `newRecordsFixture` and the other `new*Fixture` builders) build their repos with git per test instead.
- Which of these levers would pay and roughly how much, each against the measured split:
  - more fixtures on the hub template, or templates of their own;
  - shared read-only fixtures where a test only reads;
  - fewer files per fixture;
  - fewer git process starts in the code under test, such as one go-git read in place of several `git` calls;
  - merging or cutting overlapping tests, naming each candidate and the behavior it alone pins;
  - the effect of `t.Parallel` and `-p` on this package.
- Which results carry over to OSL-1046, where the report cannot measure, said as reasoning, not as numbers.

## Bounds

- Documentation only: a section in `docs/benchmarks/test-suite-timing.md` or a report under `docs/research/`, with no code, test or config change.
- Measure through `lym gate test` only, as the gate rule says, and record each command and machine state with its numbers.
- End with a ranked list of proposed follow-up tasks, each with its expected saving; file none of them.
