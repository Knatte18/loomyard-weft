---
verdict: CONTINUE
rationale: "Round 2 carries one BLOCKING finding (F1, parent-anchor-read-before-parent-placed: an unsettled parent entry's home is not where the parent lands), plus gating MEDIUM design findings F2 and F3 on keys open since round 1."
---

The decision rule gives `CONTINUE` on two counts.

- F1 is BLOCKING. It maps to `parent-anchor-read-before-parent-placed`.
- F2 (`overflow-overwrites-remembered-desktop`) and F3 (`kwin-non-current-desktop-output-unpinned`) are MEDIUM `design` findings on keys that were open in round 1.

F4 (`window-command-lost-or-gained-with-placement`) is LOW and does not gate.
The other round-1 keys are resolved in the review's `prior` list.

No finding was relabelled.
No finding cites a PATTERN entry.
The review has no `## Focus departures` section.

F3's fix runs through the run's parent: the fixer asks it to get the read-only KWin facts observed,
and to have the operator run or approve the ones that move windows.
The finding's class is `design`, so it still goes to the fixer.
