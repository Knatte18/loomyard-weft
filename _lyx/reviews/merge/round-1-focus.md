---
round: 1
focus:
  - "internal/loomcli: the parent's new `lyx loom card-pack` verb joins the lightweight wiring and is documented as mutating no state and writing only stdout and the logger. Settle whether the task's follow-on-at-start change (start entries, the shared wiring or constructors it touched) reaches that verb or its wiring, so that card-pack now brings up reed or touches follow state."
  - "internal/loomcli/doc.go: both sides added paragraphs to the same doc. Settle whether the merged text still reads true, in particular the start-entry follow paragraph against the card-pack paragraph's claim that the lightweight wiring fills only the stencils directory, and whether the 'every start entry' wording conflicts with the new verb's classification."
  - "Verb registration and hooks: the task's reed follow verb and the parent's `card-pack` verb both add loom/reed command registrations. Settle whether the merged registration, audience tags and refusal-spec rows still line up, and whether any invariant test enumerating verbs or audiences fails to cover one side."
  - "Test economy: settle whether the merge left two tests covering the same wiring (for example both sides' lightweight-wiring or command-registration tests), or left the combination of the follow change and the new verb's wiring without coverage."
---

Seed focus from the overlap in `internal/loomcli/doc.go` and the parent's new verb in `internal/loomcli`.
