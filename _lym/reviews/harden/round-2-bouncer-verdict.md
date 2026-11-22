---
verdict: CONTINUE
rationale: "Round 2 carries three BLOCKING findings (G1 archive tag back in plain refs/tags/ when the code store respells origin's URL, a reopened round-1 key; G3 the pending-teardown re-run deletes on origin what the killed child never archived; G4 both stores share one archive-tag prefix on origin) and MEDIUM G2 (reconcile still returns a raw gitrepo error), so the rule forces CONTINUE."
---

## Mapping to ledger keys

- G1 maps to `archive-tag-plain-refs-tags-one-repo`, open in round 1: reopened, BLOCKING and design, so it blocks.
- G2 maps to `push-missing-fetch-refspec-no-way-forward`, open in round 1: the same failure reaches `lym fabric reconcile`; design at MEDIUM, so it blocks.
- G3 and G4 are first raised in round 2 and are BLOCKING, which blocks regardless of key age.
- Resolved on the review's re-drive: `guard-push-bypass-any-paths`, `guard-refusal-omits-sandbox-way-forward`, `unpushed-commits-no-upstream-says-none`, `pull-missing-fetch-refspec-reports-ok`, `killed-teardown-child-orphans-unnamed`.

No finding was relabelled.

## Focus departures

All seven are ratified: each names a verb or hub property that does not exist (`hand-over` and `replay` are no `lym fabric` verbs; every sandbox hub records shortname `sbx`; no sibling sandbox root exists),
or a phase a pre-receive hook cannot hold, and the review ran the nearest drive it could and named the gap under Coverage.
The task-upsert refusal is a role limit, and notes stood in.

## Evidence

- G3's kill is simulated with `kill -9` plus a declining hook, one of the states a real crash leaves; its records half is reproduced twice (`k1`, `k7`) with a no-kill control (`k6`).
- G4's read rests on tags pushed by hand to the destination `pushArchiveTag` computes, because `ArchiveOwnerTips` needs a second machine.
  The read itself was the real verb, so the finding stands, but round 3 owes a drive that makes the tags through the real path.
- Concurrency drives (P8) carry a sequential control and a second pair of hubs and are graded.

## Fixes seen by no reviewer

The fix commits listed in the facts (G1, G4 prefix; G3, O3 pending record; G2 reconcile; O1, O2) are new text no fresh reviewer has seen, and a fix converges only a key first raised in a closing round.
Round 3 re-drives each as a claim to break.

## Coverage the campaign has not driven

- The remote warp sweep of `cleanup --apply --remote`, `ReconcileOrigin`'s ended-entry and other-owner teardowns, `ArchiveOwnerTips`, hand-over, takeover and the code-tip reset.
- A read of a hub in the middle of its clone, a long-lived process across a rewiring, and the seed killed before its first push.
- `checkout`, `pull` and `reconcile` killed at a chosen phase.
- A real host's refusal of `refs/lym/` pushes.
