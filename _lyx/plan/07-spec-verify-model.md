# Card 7 — spec-verify-model

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:** The deployed plan spec's `## verify model` stops naming this repository's Go tiers as the format's rule (D6).

- The order sentence ("The section runs, in this order, `go build ./...`, …") becomes: the section runs the hub's verify tiers in the order and form `language.yaml`'s `verify_tiers` gives, and with none set it builds the code and then runs the tests of every package a card targets.
- The `tmux` sentence after it becomes: a tier run at Publish is run by landing config's `publish_verify` where the repo sets it, and a tier `verify_tiers` says is compiled rather than run is never run by the plan.
- The rest of the section, the tier bullets included, stays, and no range `webster-fork-stencil-fixes` cites (the `Rename` row, the per-language list) is edited.

The change is static prose, which no test asserts; the spec scans in the card gate's guard step run over it.
