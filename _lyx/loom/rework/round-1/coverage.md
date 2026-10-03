# Rework coverage — round 1

- F1 — Tier 1 anomaly filing does not run under `step`: cards 11, 12.
  Card 11 calls `detectAndFileAnomalies` from `loomAfterStep` ahead of the halt note and reflection, keeps its skips, takes the entry observation after the busy probe, adds the `loomCLI.fileIssue` seam its tests pin filing through, and writes the Tier 1 line into the halt note.
  Card 12 tells the reflection stencil to file only the lyx problems behind a halt Tier 1 files, and records that ownership in `CONSTRAINTS.md` and `docs/overview.md`.
- F2 — a halt whose filing failed is lost once the run resumes: cards 10, 12.
  Card 10 keeps a failed anomaly as a pending entry in the filed-title marker and retries it on every later detection pass, pinned by a test where filing fails, the run is resumed and a later pass files it.
  Card 12 documents the retry.
- Non-blocking notes: no card needed; they confirm no new `manifest/designs/` file, `main` merged through `4d4d1bafc`, and no `gh` filing or approval gate.
