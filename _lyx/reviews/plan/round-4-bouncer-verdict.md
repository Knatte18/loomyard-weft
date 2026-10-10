---
verdict: CONVERGED
rationale: "Round 4 carries one finding (F1, MEDIUM design) on a key first raised this round, so it blocks nothing and Go carries it into Open risks; no BLOCKING finding remains, and the keys open in earlier rounds (push-drop-has-no-recorder, pull-first-drop-recordless-caller-unlogged) were not re-raised."
---

F1 is the drop's missing serialisation against a concurrent pusher's `pull --rebase`.
It maps to the new key `drop-not-serialised-against-push-rebase`, absent from the facts' keys open in an earlier round, so the decision rule's carry-over clause applies.
No finding was relabelled, and the review carries no `## Focus departures` section; its `## Focus settlements` are not departures.
