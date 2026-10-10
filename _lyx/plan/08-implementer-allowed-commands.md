# Card 8 — implementer-allowed-commands

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/claims_test.go`

**Intent:** Tell the implementer which commands are always allowed and what a tagged test gets (decision D4).
In "The sandbox rule", add: the card gate steps, the batch gate steps, `lyx loom lint-comments` and `lyx quarry resolve` are always allowed from any strand and are never a fabric reference; the sandbox ban covers mutating `lyx fabric` verbs only.
Keep the existing sentences the claims test pins ("Never clone the hub's remotes, and never run a mutating `lyx fabric` verb", `hubforge`, the stop-and-report way forward).
In "Slot-busy exits and raw `go` runs", add after the raw `go test` sentence: a `lyx gate test` step runs as written, whatever its `--tags`; a test under the `tmux` or `llm` tag that no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, and running those tiers is left to the gates after webster.
Keep "You may run raw `go test` on the batch's listed packages only" and "never a raw `go` command".
Both additions name `lyx gate test`, `lyx loom lint-comments` and `lyx quarry resolve` only, never a denied form, so the no-denied-recovery scan stays clean.
Pin the new sentences in the implementer stencil's claims in `claims_test.go`: the always-allowed list, the `lyx fabric`-only ban wording and the tagged-test compile rule.

**ImpactSummary:** Prose in one stencil and its claim pins; no Go code changes.

**Verify:** lyx gate test ./cmd/lyx -- -run 'TestNoDeniedRecovery'
