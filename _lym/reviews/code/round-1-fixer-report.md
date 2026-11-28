---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed sentence speaks of the two probes only and bounds each attempt by one remote deadline. F2 added more retried reads under the same hold, and one attempt can run several remote commands (internal/fabricengine/remoteleftover.go, probeWeftLeftover), so the sentence says each retried read and each remote command."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- **F1** (`5fa1904f0`): `TestPrimeShuttle_WiresTheReplyProbe` gives the CLI a slug with no task worktree, captures the log and refuses a "locate the task worktree" line.
  The test no longer runs in parallel, since the log capture is process-global.
  It covers the quiet absent-pair path only.
  The review's prose also names the wiring of the usage recorder for a present pair; no test covers that, and the `fix` key did not ask for one.
- **F2** (`1b65b47ce`): the retry is now `AddOptions.RetryOriginReadInTransit`.
  It wraps the lookup and fetch in `resolveWeftBranch`, Add's adopted-warp fetch, both weft fast-forward fetches and the beam-branch lookup of Worktree-Create.
  `resolveWeftBranch` takes the reader as a parameter, so checkout and reconcile still read once.
  No test was added, since F2 is not a coverage gap.
- **F3** (with `1b65b47ce`; the file is `_lym/landing/summary.md`, so the commit holds none of it): the bound on the prime-lock hold is restated, and `## What changes` lists which reads retry.
- **F4, F5, F6** (`7bd1c3913`): an `OriginNotARepository` row, a restore hook that records its error for the test goroutine, and a `wantSleeps` the `OriginAnswersAgain` row reads.
- **Self-check** (`d46fe54a3`): Add's doc and the summary name the two lookups that run once, and the CLI has a `backoffSleep` seam for the create's retries.

- **Gate finding:** a sentence in Add's doc was wrapped in mid-sentence; it is now two sentences, one per line, in a commit of its own.

Both changed packages passed `lym gate test` at tier 0 and tier 1 after the last code change.
The comment repair came after that, and `internal/fabricengine` passed tier 0 again.

## Self-check

- fix-code-correctness, LOW, `warpDefaultBranch` runs once while the text claims all reads: fixed differently; the doc and summary name it instead of wrapping it, since Worktree-Create's Add never reaches it.
- fix-code-correctness, LOW, one attempt can run several remote commands: fixed.
- fix-proposal, LOW, the beam lookup's backoff cannot be injected: fixed.
- fix-proposal, LOW, the summary claims all of Add's later reads: fixed.
- fix-consistency, MEDIUM, Add's doc claims every read is retried: fixed.
- fix-consistency, LOW, summary lines over the length limit: fixed for the lines this round added.
- fix-affected-set, MEDIUM, two reads in Add run once (`PairTaskBranchOrOrigin` under `AdoptOnly`, `warpDefaultBranch` under `AdoptLocal`): fixed; both are named in the doc and the summary.
- fix-affected-set, LOW, the per-attempt bound is understated: fixed.
- fix-affected-set, LOW, the usage-recorder wiring for a present pair stays untested: fixed as it asked, by stating it under F1 above.

## Disputed

## Deferred

## Rulings
