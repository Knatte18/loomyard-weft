<!-- This is the pack-off first step of each card in webster's implementer prompt.
     RenderForkPrompt reads it through stencilstore for a fork without a kick-start pack, and RenderRecoveryPrompt for every recovery prompt;
     the text, with this banner and the trailing newline stripped, fills the required card_read marker of webster-body-implementer, after the step's number.
     It carries no marker.
lyx-stencil: sha256=0ca0d39773fef6596a42d9a7b848e09ee8ab1fe95bf2401cd97c09099c830e28 build=1135939906fcdddd4cfdcf1bd7b46ad7ace09268 time=2026-10-12T11:39:01Z -->

Read the card file.
   It is your whole instruction for that card.
   If its `**Intent:**` field is empty, fall back to that card's one-line intent from the Card Index in `_lyx/plan/00-overview.md`, matched by the same NN/slug.
