---
round: 2
focus:
  - "internal/heddleengine/store.go `Set` and `Remove` with `refuseShipped` in shipped.go: settle whether a repository entry already stored under a slug that the shipped layer also holds can still be edited or removed through the verbs, and whether the refusal text names a step that can be followed."
  - "internal/heddleengine/shipped.go `ReadLayers` and `Layers.Ordered`: settle what a malformed heddle-lym-override.json or an override with blank text does to render, show, list and get while `lym_override` is on."
  - "internal/heddleengine/render.go `Show`, `Passages` and `passagesOf`: settle whether tests pin the override and the shipped-first order on each of the three readers, not on `Render` alone."
---

The focus names where the round 1 gating findings sat; the reviewer judges the current code afresh.
