# A shuttle hold shows as a wait mark and in the loom status waiting line

`hold-wait-mark` · Note · enhancement, shuttle

From run-recovery PR-Gate (2026-10-07): run-recovery made every wait visible (reed `@lyx_wait` mark, `waits` token, `lyx loom status` waiting line) for verify, gate and background-shell waits. shuttle-turns adds a new hold state, a turn end without output that holds; unless it sets a wait mark too, a held session looks like a hang. Wire the hold into the same marker files once both have landed.
Also from shuttle-turns PR-Gate (2026-10-07): a gated run whose last event is a waiting turn end with every output file present can sit up to `background_shell_wait_min` (10 min) before it finishes Done.
