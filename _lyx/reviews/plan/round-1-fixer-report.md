# Plan-Review round 1 — fixer report

All seven findings fixed.
The `approved:` key in `00-overview.md` was not touched.

## Changed

- F1 (MEDIUM), `04-spec-resume-session.md`: added a `Bound:` paragraph to the Intent.
  It says the engine checks only the id's shape and lets a run take over any well-formed session id, that card 5's `SessionResumer` preflight and card 6's transcript and live-holder check bound it, and that `--adopt` is the only production caller.
- F2 (MEDIUM), `03-prompt-suggestion-off.md` and `13-orch-live-smoke.md`: added `internal/orchengine/doc.go` to card 3's Edit list.
  Card 3 now replaces the orchengine prompt-suggestion residual in the same commit as the settings change, and card 13 no longer does.
- F3 (LOW), `07-orch-config-keys.md`: rewrote the ImpactSummary.
  It now says every orch verb fails to load an existing `orch.yaml` until it is reconciled, names the fixtures that must carry the keys, and confirms no other module reads the config.
- F4 (LOW), `13-orch-live-smoke.md`: the stop-and-report rule now records the outcome in Open risks.
  It names the decision record's fallback as the follow-up fix (resume without a positional prompt, then send the one-line adopt stencil once startup clears) and says why this card does not apply it.
- F5 (LOW), `13-orch-live-smoke.md`: `TestSmokeOrch_SoftCycle` now requires a `soft_idle_s` of tens of seconds, so the background-shell turn lands before the start prompt's quiet window closes and that turn's end starts the soft cycle.
- F6 (NIT), `11-orch-spec-bypass-agents.md`: rewrote the ImpactSummary.
  It now gives `orchShuttleConfig`'s one caller and one test, and says that `orchSpec` is the only spec orch starts, so every orch-launched run still gets the Agent tool.
- F7 (NIT), `12-orch-start-adopt.md`: the Intent now names both test fakes, `fakeStarter` and `starterFunc`.
  The ImpactSummary now counts the seam's one production implementer and two test fakes.

## Deferred, with reason

None.
