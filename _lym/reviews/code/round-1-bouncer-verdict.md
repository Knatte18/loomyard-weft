---
verdict: CONTINUE
rationale: "Round 1 carries a BLOCKING design finding (F1, --unset against the reconcile-only removal rule in PATTERN-config-layers) and a MEDIUM design finding (F3, untested refuse_raw fallback); no relabels."
---

F1 cites a PATTERN entry the diff violates without a `Pattern-Departure:` trailer, so it stays BLOCKING.
F3 is a MEDIUM design finding, so it gates too.
F4 is a MEDIUM `decision` finding, not gating by class, and is deferred to the parent's call on re-keying the reinterpretation.
No Focus departures section was present, and no finding was relabeled.
