---
verdict: CONTINUE
rationale: "Round 2 carries BLOCKING F1 (consistency, a surviving \"(detached)\" literal against PATTERN-named-constants), so the loop continues; F3 reopens the launcher-only origin key at LOW, F2 and F4 are new keys at MEDIUM, no relabels."
---

Round 2 has one BLOCKING finding, F1, which is gating regardless of class because the decision rule continues on any BLOCKING finding.
F1 cites `PATTERN-named-constants`, a listed entry, so it stays BLOCKING.

Mapping to ledger keys:

- F1 is new: `detached-literal-survives-next-to-constant`.
- F2 is new: `prefix-changed-between-nonempty-values-unrecognised`, class `decision`, deferred to the parent.
- F3 reopens `remove-origin-lookup-launchers-and-double-ask`, since it concerns Remove's origin lookup skipping a pair whose only local remnant is its launcher directory.
  It is LOW and not a gating MEDIUM, so it does not block on its own.
- F4 is new: `unresolvable-records-root-skip-untested`, class `scope`.

No finding is relabeled and no focus departures are present.
The nine other round 1 keys do not recur in round 2 and are carried forward as resolved.
