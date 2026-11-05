```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checking that every fact the fix added or rewrote holds against the code at HEAD.

no findings

The claims I checked all hold:
- **D6:** `AgentSessionOpen` and the interrupted-agent probe exist in `internal/driftengine/records.go:278` and `internal/driftengine/verify.go:125-140`.
- **D6:** the run status the drift deps read is `shedengine.Status`, read in `internal/loomcli/driftwiring.go:60-69`.
- **D14:** a gate slot is taken through `Pool.Acquire(ctx, Holder)`, and `Holder` carries no gate key, so D7's capture can hold a slot without one.
- **D7:** `lym loom goto` is an `AudienceOperator` verb, which covers "the orch acting for them" (`internal/clihelp/annotations.go:22`). Re-entering Darn-Write respawns the writer (`docs/overview.md:552`).
- **D15:** a retired `CategoryNone` key is admitted and dropped on load (`internal/configengine/doc.go:76-78`), so "seeds, and the load drops the key" holds.
- **D8:** Master edits no code and answers a gate failure through a fixer fork or `fix-verify` (`contracts/stencils/webster/webster-template-master.md:20`, `:192-198`).
- **D8:** gate keys hold only test tiers and the `comments` and `pattern` lints (`internal/gateslot/config.go:424-433`).
- **D14:** `goReadGates` and the gateslot template name `drift-verify` (`internal/gateslot/tiers.go:113`, `internal/gateslot/template.yaml:30`).
- **D11:** the claim that the caller check guards only a mistaken call matches `internal/loomcli/decision.go:11`.

Tool calls:
- One Bash call read `internal/loomcli/driftwiring.go:55-70`, `gateslot.Holder` and the `AudienceOperator` definition. My context did not show the status-read seam, the slot holder's fields or who may run `goto`.
