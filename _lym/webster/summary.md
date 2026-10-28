# Creel delivers every post to a Claude strand through hooks

All four batches (18 cards) recorded `done`; no recovery, no `Pattern-Departure` trailers and no renames in any digest.

## Batch 01 — reed-creel-hooks-mark (cards 1–6)

Head `fe3e96dd`.
Built the reed `CreelHooks` mark and its launch path, quoted sender names in creel lines, the 8 KiB line cap on post and claim, LOOM's decision notice fitting that cap, and the claim's boundary with the 6-block Stop cap.
Deviations (files outside the batch's targets): `internal/creelengine/deliver_test.go`, `internal/creelengine/verbs.go`, `internal/loomcli/decision_test.go`.

## Batch 07 — shuttle-idle-no-turn (cards 7–10)

Head `72cb7106`.
Built the idle probe's turn reading, the delivery seam fields separating hook strands from pane-mail strands, the pane path restricted to strands without hook delivery, and the daemon's once-per-claimable-set waiter ring.
Deviations: `internal/creelcli/delivery.go`, `internal/creelcli/delivery_test.go`, `internal/creelengine/pool.go`, `internal/creelengine/verbs.go`, `internal/testkit/creelfake/creelfake.go`.

## Batch 11 — creel-bell (cards 11–14)

Head `d644b04d`.
Built the typed bell, the shuttle hook capability, the `hook-read` and `hook-wait` verbs, and Claude's rendered settings merging the Stop delivery and adding the waiter.
Deviations: `internal/creelcli/cli_test.go`, `internal/creelcli/hookread.go`, `internal/creelengine/deliver.go`, `internal/shuttleengine/engine.go`.

## Batch 15 — creel-post-interrupt (cards 15–18)

Head `3c429976`.
Built `post --interrupt`, the trust sentence through one prompt file, the hook-delivery docs, and a real-Claude smoke test.
Deviations: `internal/creelcli/smoke_llm_test.go`, `internal/creelengine/protocol.go`, `internal/creelengine/server.go`, `internal/creelengine/verbs.go`.

## Scope guard

Every digest carried `status: done` and empty `warnings`; the deviations above are informational.
The plan-level `webster` gate runs after this turn ends.
