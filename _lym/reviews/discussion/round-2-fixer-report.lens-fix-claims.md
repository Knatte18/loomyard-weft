```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

1. **MEDIUM.** D4's new weft bullet says: "The way forward is to wait for the pair's teardown to finish, or finish it with `lym fabric remove <slug>`, then re-run the upsert."
   - **Problem:** `lym fabric remove <slug>` as written does not end the liveness that matters. Origin's weft branch, which is what keeps a torn-down pair live, is deleted only with `--remote`. And once the detached child has deleted the local pair, the command has no pair to act on.
   - **Evidence:** `internal/fabriccli/fabric.go:165-195`, where the Long text says "--remote also deletes the weft branch on the weft remote" and that a removal without it does no remote branch step.
   - **Also:** `internal/battencli/wire.go:415-416` says "The removal hands its remote steps to a detached child … the child mails the orchestrator a failed step with its way forward". A failed remote step therefore has its own way forward, and the record does not name it.

2. **LOW.** D3's new sentence: "or when the board already holds the filing as `boardengine.Board.HoldsRaddleFiling` judges it."
   - **Problem:** the record does not say where a loom verb gets the slugs that seam needs.
   - **Evidence:** `internal/boardengine/raddlefile.go:407-411` has the signature `HoldsRaddleFiling(sketch string, slugs []string)`, documented as "With no slugs it reports false". It judges a filing only from the part slugs, which `add`, `replace` and `store` can read only from the run's proposal.

Every other changed claim I checked holds:
- Record-Write never overwrites a record (`recordwrite.go:124`).
- `IdentityText` strips only carry-over blocks and added entries (`identity.go:11-16`).
- Drift gives added entries precedence (`seats.go:104`), and darn gives it in its gate and scope (`darnwrite.go:47`, `darn doc.go:64`).
- `decision.go:9` states the mistaken-call limit.
- The liveness test matches `add.go`'s doc.
- `internal/shellguard` has no `git commit` deny.

Calls made:
- Grepped `fabriccli` for the remove command and `pairteardown`/`battencli` for the detached teardown, to check the new D4 way forward and the support-log teardown claim. My context lacked both.
- Read the `remove` usage, the `finishRemoval` doc and the `HoldsRaddleFiling` doc. My context lacked them, and the new D3 and D4 text relies on them.
- Read the `lym fabric remove` Long text, to check whether it finishes an in-flight remote teardown. My context lacked it.
