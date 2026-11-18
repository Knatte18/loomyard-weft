```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether every sibling decision, criterion and cross-reference that the change set touches still agrees with it, under the fix-consistency lens.

**LOW — the support log's Turn 3 rotation line is stale**
- Changed text: D3 now says the perch "expands the reviewer and fixer lists cyclically to its hard cap, `hard_max_bounces` (D9)… a round a parent's continue runs past `max_bounces` included."
- Problem: `support-log.md` Turn 3 still says the lists expand to the round cap.
- Evidence: the first interactions/walkthrough bullet reads "Fixed: D3's perch expands the lists cyclically to the cap; criterion added." The fixer corrected the Turn 3 line for F1 but left this one, so the log now disagrees with D3.

**LOW — D6's facts rendering misses the deferred findings the live gate writes**
- Changed text: D4 now says that when `live_attempts` run out, "each failing test is recorded as a deferred finding, so the Bouncer's carry-over and the judge's CONVERGED rule read both (D6)."
- Problem: D6 says the facts rendering lists only the deferrals the fixer makes, and D6's third judge bullet carries over only those, "deferred by the fixer". So D4's gate-written deferrals have no stated reader in D6.
- Evidence: D6 says the rendering "also lists the findings the fixer left deferred". The new D4/D6 acceptance criterion ("the next Harden-Bouncer's facts carry both") expects them to be there.

**LOW — two wordings of the halted-campaign reap disagree**
- Changed text: D5 Bounds says "a campaign halted capped or circling runs it as it halts". The new criterion says "Finalize, and a campaign halted capped or circling, reap the sandbox root before teardown".
- Problem: these describe two different trigger points, at the halt versus before teardown, for the same reap.
- Evidence: the two quoted lines.

I made no further tool calls.
