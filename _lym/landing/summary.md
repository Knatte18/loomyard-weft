# Survey of where fabricengine's test time goes, with ranked cuts

A documentation-only change: one research report, `docs/research/fabric-test-speed-survey-2026-11-03.md`, and a pointer section at the end of `docs/benchmarks/test-suite-timing.md`.
No code, test or config changes, and no board task filed.

## What the report holds

- Timed `tier1` and `tier0` runs of the package through `lym gate test` on neve, with the commands, the gate's build and vet share, the load and the held slots at every run, and a `-parallel` sweep at 1, 4, 12 and 24 run twice.
- The cost of every top-level test from a serial run, folding parallel subtests into their parent, and the same per file with the fixture builder each file uses.
- Every git process of a run from git's trace2 event files, split by subcommand, by spawner and by the fixture marker, and tied to a test through its temp-directory path; and every process start and file operation of a run from `strace`, split by process type and path class.
- A catalogue of the package's fixture builders: which use `CopyHub`, which still clone through `NewHub`, what each layers on a template, and the slugs that keep the pair-carrying templates unused.
- Six levers judged against the measured split, a table of sixteen merge-or-cut candidates each naming what it alone pins, and a ranked list of nine follow-ups with expected savings.
- Reasoning for OSL-1046, where Cortex XDR holds each new binary and scans every file opened: process starts and file opens are the counts that carry over, and one trace2 run there would rank the levers.

## Design decisions

- The report lives under `docs/research/` as a dated snapshot, in the form of the other reports there, rather than as another section of `docs/benchmarks/test-suite-timing.md`, which already runs to 800 lines; the timing document gets a short section that points at it, so the recurring audit finds it.
- Timings are presented with their load and as upper bounds: other worktrees' gate runs held slots throughout, and the same command spread about 30 percent between runs.
  The counts of processes and files are load-independent and are the report's firm numbers.
- Git processes are counted through git's own trace2 facility, the route `cmd/testtiming -resources` already uses, set in the environment of a `lym gate test` run; file counts come from `strace` around a separate run, so the timing runs stayed uninstrumented.
- The merge-or-cut candidates are read from the code, since the committed redundancy report lists no candidate for the package; each row names the behavior the candidate alone pins so a fold keeps it.
- The operator's note during the run, that Cortex XDR scans every file opened, is folded into the OSL-1046 section as the premise of its reasoning.
