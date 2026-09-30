Seen 2026-09-30 for quarry-cli-answers and status-strand-by-driver: after landing and `fabric remove --remote`, their reed sessions were still alive hours later.
A driver that wrote its stop report after the removal recreated `<hub>/<slug>/.lyx/...` and `<hub>/<slug>/_lyx/shed/<slug>/drive-reports/<report>.md`: a stray directory, and a drive report that never reached the archive.
The hub killed the sessions and deleted the dirs by hand.

Fix: remove (and batten teardown) ends the pair reed session (reed down) before deleting the worktrees, and waits for or refuses on a live driver; a stop report written after done should land before the archive tag.