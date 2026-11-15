# Build the adopt redesign in a beam: run FSM, push before act, Bolt holds, foreground adopt

`adopt-build` · Project · feature, batten, tender, dobby, orch, fabric, loom

The build tasks raddle split from the Sketch `adopt-redesign`, run in a beam so they land on `main` together (operator, 2026-11-15).
No state between two members landing reaches `main`, so no installed lym runs half of the redesign.