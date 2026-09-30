# Hub orchestrator as a reed strand with automatic handoff

All 14 batches reported `done`, and the plan-level integration verify reported `status: OK` at HEAD `cbb60846c`.

- **Shuttle seam (1–3):** `shuttleengine.SessionCycler` plus `Runner.ReadEvents`, `ContextTokens`, `SessionIdle` and `ClearSession`. `claudeengine` reads context tokens from the transcript and classifies an idle pane. It also plays `/clear`. The pane fixtures are hand-written, not captured live.
- **orchengine (4–9):** degrading config registered in `configreg`, three stencils and their renders, and persisted state. It also holds the start decision and launch-prompt choice, the four-phase cycle `Watcher`, and the single-instance watch loop. The loop's tick-error cap is 30 and its watch-lock retry is 5 attempts at 200ms; both were chosen by the forks because the cards left them open.
- **orchcli (10–12):** `lyx orch` with prime-only resolution and the `status`, `cycle`, `stop`, hidden `watch` and `start` verbs, registered in `cmd/lyx`. A `reedUp` seam was added beside the two the card named, and `orch` was added to the sandbox suite's excluded modules.
- **Docs (13):** `internal/orchengine/doc.go`, plus updates to `docs/overview.md`, `CONSTRAINTS.md` and `manifest/roadmap.md`.
- **Smoke (14):** `TestSmokeOrch_OneFullCycle` compiles under `go vet -tags smoke`, but it was never run, since it needs a live billed Claude session. The open risks in the doc stay "unverified, pending a smoke run".

No batch reported deviations from its declared file-ops. Batch 1's record carried informational out-of-glyph warnings only.
