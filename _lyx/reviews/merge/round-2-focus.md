---
round: 2
focus:
  - "In internal/shuttleengine, `Send` and `Interrupt` on a live strand that no run record names (`recordlessSendContext` in send.go, the recordless branch of `Interrupt` in run.go) now resolve the engine from the strand's provider tag, which `Open` (open.go) stamps from the engine it spawned on. Settle whether a strand tagged with a non-default engine reaches that engine and an empty tag still reaches the default, whether reed's `StrandStatus` and `requireLiveStrand` carry the tag correctly, and whether the `Send_and_Interrupt_on_a_live_strand_with_no_record_consult_the_default` test in enginedispatch_test.go covers the tagged case."
  - "Settle whether the merged tree still upholds `PATTERN-shuttle-provider-seam` and `PATTERN-leaf-packages` after reed's `StrandStatus` gained a provider field: provider specifics may live only under `internal/shuttleengine/<engine>`, and reed's import set must stay within its leaf-package allowance."
---

The focus targets the new text that resolves the engine for recordless sends, since no fresh reviewer has seen it.
