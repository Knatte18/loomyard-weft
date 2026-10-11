---
round: 2
focus:
  - "Error stops in `_lyx/discussion/decision-record.md` D1, D6 and D7: `internal/shedengine/run.go`'s `appendHistory` appends nothing when a producer returns an error with no outcome, so a recurring `producer` or `bootstrap` error has the same producer, history length and `failed` state before and after a re-launch. Settle whether the attempt key and the same-stop rule, as the decisions now read, let the verb tell an unapplied `resume` verdict (served on a D6 wake) from one already applied and recurred (handed on), and whether one key is named once."
  - "Un-pause in D6: the driver un-pauses an `awaiting` or `paused` state the arbiter's records left, and `lyx loom goto` and `lyx loom pause` also leave a run paused. Settle whether the decision bounds the un-pause to state the arbiter itself left, so that a pause or goto set by the orch or operator during a live arbiter is not undone, and whether the Open risks section only describes what a Decision should bound."
  - "Attack surface of D2 and D3: the repair fork's escalation is kept only by the driver stencil, and Go cannot see it. Settle whether D2 states what an error stop reaching the verb without the fork can now skip, and what bounds that (spawn budget, same-stop rule, the arbiter's lack of repair verbs)."
  - "Watched marker in D2: `internal/battenshed` logs and carries on when `MarkWatched` cannot write the marker, so a live batten can watch a run whose marker is absent. Settle whether D2 states the bound for a missing marker, not only a stale one, at the batten-owned Publish halt."
---

The focus entries name where to look; they do not cap severity or state a verdict.
