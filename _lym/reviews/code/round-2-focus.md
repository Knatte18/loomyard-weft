---
round: 2
focus:
  - "internal/boardengine/render.go runFieldLines: check that the `pull_request` and `on_land` line keys come from named constants, and that no other site in internal/boardengine (replay.go, front.go, store.go) still repeats either key name as a bare string literal outside struct tags."
  - "internal/boardengine/doc.go: check that every sentence about what the README entry cell holds agrees with `entryCell` and `recipeCell` in render.go, in particular where the profile is rendered."
---

Both entries target the text the round-1 fixes changed, which no fresh reviewer has seen.
