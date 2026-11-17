# Survey the mutation-confirmed redundancy candidates across every package

`redundancy-confirmed-survey` · Task · research, testkit, gate · running · Code-Bouncer

## Goal

Task `redundancy-judge-asserts` (`e97e2474`) made `lym gate redundant` confirm each coverage candidate by mutation:
a test stays listed only when its covering tests kill every mutant it kills.
Before the cut `redundancy-sweep-existing` relies on that list, this survey checks it across the repository, as `redundancy-candidates-survey` (`194650f0`) did for the old judge.

## The runs

1. Build the maps in this task's worktree with one `lym gate testmap` over every package, as the earlier survey did.
2. Run `lym gate redundant` once on each package that has a candidate, and record per package:
   candidates, confirmed, refuted, not judged with its reasons, and wall time.
3. Read a sample of exactly 30 confirmed candidates, spread over the packages with the most confirmed ones,
   using subagents; for each, judge whether it checks something its covering tests do not.
   Name the kind of check the mutator missed for each one it did, such as a check on another package's result.

## Read

- How many confirmed candidates in all, and their total run time: what the cut could save.
- The share of the sample that is a false confirmation, and which kinds of check cause it.
  That share decides how much `redundancy-sweep-existing` must read before it cuts.
- Packages where the verb is slow or a mutant hangs a test until its bound (`internal/loomcli` took 8.6 minutes in the landing's measurement).

## Bounds

Change no production code and no test, and nothing under Bolt.
The report goes under `docs/research/speed/`.
