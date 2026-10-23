# Card 38 — driver-stencils-hand-back

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils/shed/shed-template-driver-guide.md`

**Intent:** An LLM driver neither starts the binary watch for a landing halt nor re-steps it after a build change (D11).
The driver stencil is recipe-blind and branches only on envelope fields, policy words and error kinds, so it reads the `hand_back` mark card 34 added.
Every site that re-steps on `binary_changed` skips a stop marked `hand_back` and hands it back: the stencil's Re-launching bullet on `last_step.binary_changed`, its Parking step 3, which starts the binary watch for a binary-change-eligible stop, and the guide's Binary watch section, which re-steps only when `binary_changed` is true, the clean-tree guard passes and `hand_back` is false.
A stop marked `hand_back` is handed back to the parent with no tackler, arbiter or re-step, since only the orch's `lyx loom resume` lifts it.
The stencils name no row; the Go-side tests of the mark stand behind this prose, which no test pins.

**ImpactSummary:** Prose in two stencils; the mark they read is card 34's, and no other stencil branches on `binary_changed`.
