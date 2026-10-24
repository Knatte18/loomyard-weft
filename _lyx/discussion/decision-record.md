# Decision record — idle-notice-no-live-run

## Goal

Find why Batten's activity reader saw no live run while `loom-tail-review`'s Plan writer worked, and close the gap or make the next occurrence show its cause.

## Scope

- The notice text "no agent activity readable" comes only from `battenshed`'s `judgeAgents` branch that `shuttleengine.ReadAgentActivity` reaches with zero readings.
- `ReadAgentActivity` drops a run directory on four paths: an unreadable record, an absent record, a record that is not live (outcome not `running`, or its waiter pid absent or dead), and an engine the set does not hold.
  The unreadable-record and unknown-engine paths already log once per episode; the not-live paths log nothing.
- The incident's records cannot settle which path it took.
  `loom-tail-review` ran on machine `osl-1046` from 12:18 to 14:05 local on 2026-10-23, and its run directories, trace logs and that day's creel journal are not on this machine; the weft repository holds only its tracked records, which carry no shuttle run state.
- In scope: a once-per-episode log line in `ReadAgentActivity` for a record that reads `running` while its recorded waiter process is gone, with a recovery line when the run reads live again or ends.
- Out of scope: changing which runs count as live, the notice text, and Batten's quiet judgement.

## Decisions

- **Log a running record whose waiter is gone, once per episode.**
  `ReadAgentActivity` keeps skipping such a run, and it now writes one Warn naming the run directory, strand and pid at the first skip, Debug lines while the skip repeats, and one Info when the record reads live again or no longer reads `running`.
  Rationale: of the silent paths, a waiter that died while its agent keeps working is the one that hides a working agent, and a line in the reader's caller's trace tells the next occurrence apart from a record that genuinely ended.
- **A separate episode set keyed by run directory.**
  Rationale: the unreadable-record episode recovers on every readable read of the same run directory, so sharing its key would end the new episode on each poll and repeat the Warn.
- **A record with no pid is not logged.**
  Rationale: it predates the pid field, so a missing pid says nothing about a lost waiter.

## Constraints

- `PATTERN-log-levels`: the read runs in Batten's and the loop watchdog's polls, so a repeating condition logs once at Warn, then at Debug.
- `PATTERN-tolerant-records`: no change to the run record's shape.
- `PATTERN-shuttle-provider-seam`: the change stays in the provider-neutral reader.

## Auto-mode assumptions

- The board entry's quote is taken as possibly exact; the incident cannot be replayed, so the change instruments the gap rather than claiming a fix.
- A Warn per waiter-gone run per process is acceptable noise: such a record is an anomaly, and the recovery Info pairs it once a new waiter attaches.

## Open risks

- If the incident took a different silent path, such as a record already reading a terminal outcome while its agent kept working, this change does not show it; the absence of the new Warn in the next occurrence's trace points there.

## Acceptance criteria

- A run reading `running` with a dead recorded waiter is still skipped, and repeated reads log one Warn and Debug lines for the rest of the episode.
- Once the record reads live again, the next read logs one Info recovery line.
- A record with no pid and a finished record log nothing new.
