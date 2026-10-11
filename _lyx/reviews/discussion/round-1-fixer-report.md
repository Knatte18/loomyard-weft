# Fixer report — discussion, round 1

Every finding was checked against the code and found valid; all seven are fixed in `_lyx/discussion/decision-record.md`, with the new picks recorded in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1** (MEDIUM, design): checked `contracts/stencils/loom/loom-template-discussion.md`, whose "What you may write" section names the two files as the only writable ones.
  D2 now adds a second optional marker inside that list, rendering the ready-marker path for `twill` and nothing for `single`, so `single` stays byte-identical and no later section contradicts the directive.
  New Acceptance criterion: the rendered twill write list names the two files and the marker, and no other path.
- **F2** (MEDIUM, consistency): D3's marker list now carries the host steps holding the format and self-check (Step 5, Step 6) and the self-check command (`lyx loom validate-discussion`).
  D4 is phrased host-neutrally, with the Discussion values in parentheses as the rendered values, steps 1, 3 and 7 included.
- **F3** (MEDIUM, design): confirmed in `internal/shuttleengine/claudeengine/settings.go` that the fork-only hook greps the payload for `"subagent_type":"fork"` from any session.
  D6's bound now says the hook admits a fork from inside a fork, that a fork's read-only and no-Agent-call rules are stencil-level only, that the downstream gates read only the two discussion files, and that nothing but the stencil bounds a fork's write outside them; it notes shuttle's `ForkAudit` records fork writes, Agent calls and Bash commands with no twill policy reading it.
- **F4** (MEDIUM, design): `pattern/PATTERN-completion-signal.md` confirms reed's resume relaunches a gated strand without a prompt.
  D4 now states that a relaunched writer records each unreturned fork as "no report" in step 7, finalises from the findings it holds and never re-forks; D7 states the marker survives a relaunch harmlessly, since no forks are spawned to wake.
  The directive wording criterion now pins this.
  Recorded as Q17 in the support log.
- **F5** (MEDIUM, design): `pattern/PATTERN-daemon-wakeups.md` exempts only bounded waits under a minute and names `internal/fswatch` as the one file-event source.
  D9 now has the shared poll stat once a second, at the floor, states the per-call cap as the deadline that keeps it outside "What counts", says no `fswatch` watcher is used because a watched path's parent may not exist yet, and says `await-review`'s poll moves from 200 ms to 1 s.
  The Constraints bullet is rewritten to match.
  Two rejected alternatives (keeping 200 ms, building on `fswatch`) and Q18 are added to the support log.
- **F6** (LOW, consistency): D9's opening drops the operator attribution and the caller enumeration and says "any agent ... while it waits on a file"; the neutrality requirement stays.
  The provenance remains in the support log (Interview turn 10, Q15).
- **F7** (LOW, scope): added the criterion that the unrendered directive stencil names no Discussion file, section, step or verb; the write-list criterion from F1 covers D2's marker-only extension.

## Disputed

None.

## Deferred

None.
