---
round: 1
focus:
  - "Constraints `PATTERN-test-speed` and `PATTERN-daemon-wakeups` prescribe PATTERN edits (a rewritten background section, a removed line); settle whether the record states each as a PATTERN change the spec's `## PATTERN changes` section carries, and whether any prescribed overview line is a change at all."
  - "D4 and D7 hold the tier's gate slots and per-test memory caps while the coverage builds run outside `memory_mb`; settle whether each bypass of a bound (builds outside the cap, `slots: all` held for the whole call) states what it lets through and what limits it."
  - "D5 reruns a failed, timed-out or killed test alone and lets that outcome stand; settle whether the rerun can mask a real flaky or order-dependent failure as a pass beyond the list the report prints, and whether that is bounded."
  - "D6 treats a dirty package as unmeasured while D7 says the run still writes every map it measured; settle whether the two agree for a package built from files that change between the pre-build and pre-write checks, and whether any fact the design rests on (what counts as a linked in-module package, how HEAD is read) is left for Plan to find out."
  - "D1 makes `--map` part of the detached call's key and refuses several flags; settle whether each refusal and the detached keying is reachable from an acceptance criterion and bounded, and whether any criterion is untestable prose not opened with `Review:`."
  - "Support log's Turn 3 lists findings already fixed; settle whether any content in `decision-record.md` is deliberation narrative that belongs in `support-log.md` (for example the Rationale and Bound text of D2, D4, D5, D7), extracting any requirement first."
---

Seed focus from one read of both artifacts and the discussion spec.
No entry caps severity or states a verdict.
