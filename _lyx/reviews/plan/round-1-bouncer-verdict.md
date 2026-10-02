---
verdict: APPROVED
rationale: "All seven round-1 findings are fixed in the cards (card 4 Bound paragraph, card 3 now owns the orchengine prompt-suggestion residual, ImpactSummaries of cards 7/11/12 rewritten, card 13 stop rule names the fallback and soft_idle_s timing); every Decision and Constraint maps to a card, verify runs every targeted package with -tags integration, and every new edge states its bound."
---

The review's verdict was `APPROVED` with seven findings, and the fixer report says all seven are fixed.
I checked each fix against the card text:

- F1: card 4's Intent now has a `Bound:` paragraph.
  It says the engine checks only the id's shape, and it names card 5's `SessionResumer` preflight and card 6's transcript and live-holder check as the bound.
  It also names `--adopt` as the only production caller.
- F2: card 3 now lists `internal/orchengine/doc.go` and replaces the prompt-suggestion residual.
  Card 13 no longer touches that residual.
- F3, F6, F7: the ImpactSummaries of cards 7, 11 and 12 now state blast-radius conclusions.
  Card 7 says every orch verb fails until `lyx config reconcile` runs.
  Card 11 gives `orchShuttleConfig`'s one caller and one test, and says `orchSpec` is the only spec orch starts.
  Card 12 counts one production implementer and two test fakes.
  Card 12's Intent also names both fakes, `fakeStarter` and `starterFunc`.
- F4: card 13's stop-and-report rule names the decision record's open-risk fallback as the follow-up fix.
- F5: `TestSmokeOrch_SoftCycle` now requires a `soft_idle_s` of tens of seconds.

The review has no `## Focus departures` section, so there are no departures to ratify or reject.
I agree with its verdicts on the round-1 focus entries.
Card 11's removal of `orchShuttleConfig` is licensed by Decision 3's per-run allowance.
Card 9's `ResetForFreshLaunch` keeping `CycleTrigger` and `LastDeferral` follows that function's existing rule of keeping every history field.

My own pass against the rubric found no new issues:

- **Fidelity.**
  Each Decision is carried by at least one card:
  1 by card 1; 2 by cards 7 and 11; 3 by cards 2 and 11; 4 by card 3; 5 by cards 4, 5, 6, 8 and 12; 6 by cards 7, 8, 9 and 10; 7 by card 13.
  The Constraints are carried too.
  The provider seam stays inside claudeengine, and the `SessionResumer` doc names no provider.
  The geometry stays told: card 6 uses `claudeProjectDirFor` and `claudeHomeDir`, which the fork audit already uses (`audit.go`), and it tells `checkResumable` the registry directory.
  Config strictness is kept by the template keys.
  Spawn observability is covered by card 12's `Info` log, and CLI help by card 12's helptree assertion.
  The cycle guarantee is restated in card 9.
  Each card's docs land with its behaviour.
  No card adds scope the record does not license.
- **Anchors.**
  The symbols the cards name all exist in the tree:
  `paneCwd` in `run.go`; `buildLaunchCmd` and `buildResumeCmd`; `denyInstalls` and `buildDenyNotice`; `settingsDoc`; `orchShuttleConfig` and its test in `cli_test.go`; `fakeStarter` and `starterFunc`; `statusFields`; `ResetForFreshLaunch`; `tickIdle` and `tickHandoff`; `phaseEvents`; and `Event.Message`.
  `lyx shuttle run` already returns `sessionId`, which card 13's adopt test depends on.
- **Verify coverage.**
  `## verify:` runs `go test -tags integration` over shuttleengine, orchengine, orchcli, contracts/stencils and cmd/lyx, which covers every package a card targets.
  The smoke tag is compiled with `go vet`, and card 13 runs it in its own `**Verify:**`.
- **Attack surface.**
  Each new edge states its bound in the card that introduces it:
  `PermissionMode` in card 1, `AllowAgentTool` in card 2, `ResumeSessionID` in card 4, the warning-and-proceed registry path in card 6, the soft trigger in card 9, the dropped config rewrite in card 11, and `--adopt` in card 12.
- **Granularity and typing.**
  The plan has no `Custom` cards.
  Its cards split into independently testable units.
