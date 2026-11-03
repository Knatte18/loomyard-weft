---
format: 5
approved: false
language: go
---

# Plan: pawl checklist in a second overview window of a beam's session

While a beam's pawl run lives, the beam's session shows the run's checklist in an overview window of its own, named for the recipe, right after the beam overview window, whichever of `lym loom start` and `lym beam start` adds its strand first.
reed learns an overview order key and admits several overview strands (D1), its focus fallback takes the lowest-index overview window (D2), the runs strand stops replacing the loom status strand (D4), and the status strand always takes the overview anchor one place after the runs strand, titled with its recipe (D3), which heals an existing session at the pawl run's next loom start (D5).

## Card Index

1 — reed-overview-order-key — Display.Order, overview placement by order key, the one-overview refusals removed and an order change refused on update
2 — reed-overview-focus-fallback — relevantWindow and followTargetWindow fall back to the lowest-index overview window
3 — runs-strand-beside-status — the runs strand takes order key zero and is added beside a loom status strand, never in its place
4 — status-strand-second-overview — the status strand always takes the overview anchor after the runs strand, titled with the run's recipe

## Shared Decisions

- The order key is `render.Display.Order` (`order,omitempty`); an absent key reads as zero, so displays an older lym recorded sit at the front with no migration.
- Placement reads each overview window's key from reed state, mapping each overview strand's bound pane to its window, never from a window option, so `@lym_overview` stays a plain marker that the engine-less follow rule reads.
- The two order keys are `orchcli.RunsStrandOrder` (zero) and `loomcli.statusStrandOrder`, derived from it.
