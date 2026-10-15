---
verdict: CONTINUE
rationale: "Round 2's only finding, F1 (MEDIUM, design), is the round 1 failure of an unreconciled local TTL key refusing every shuttle.yaml load, still open because D4 leaves the retired-key category and open-map mechanism unspecified; it maps to the key stale-local-ttl-key-refuses-shuttle-load, open in round 1, so it is a gating reopened key."
---

Round 2 carries one finding, F1, class `design`, severity MEDIUM, unrelabeled.
It argues that D4's retired-key rule cannot be implemented as written: the five keys cannot stay in `Categories` or `OpenMaps` once they leave the template, and an exact-name check misses `claude_prompt_cache_ttl_roles.<role>` units.
A plan writer following D4 could therefore reproduce round 1's F1 failure, so I map it to `stale-local-ttl-key-refuses-shuttle-load`.
That key was open in round 1, and the finding says the earlier fix is incomplete, so the key is reopened and the decision rule gives `CONTINUE`.

The five other round 1 keys were not raised again, and the review's Rejected section settles the sparse-layer collision (focus 2) and the D8 empty-set guard (focus 3).
I carry them as resolved.
The review has no `## Focus departures` section, so there is nothing to ratify or reject.
No finding cites a PATTERN entry.
