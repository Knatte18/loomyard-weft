---
round: 1
focus:
  - "Fidelity, card 1: decision D6 says `tmuxkit.Main` unsets `LYX_REED_TMUX_CONF`, but the card edits `tmuxkit#setEnv`. Settle whether `Main` reaches `setEnv` on the path that guards every tagged package, so the unset covers the leak D6 names."
  - "Fidelity, acceptance criteria against card 3's `TestBoot_TmuxConf` and `TestServerSpawnArgv` rows: settle that each criterion has a covering test, in particular the empty-key hermetic marker, a second worktree not re-sourcing, `-f /dev/null` kept on the spawn and probe argv, and Windows sourcing nothing."
  - "Fidelity, card 4: settle that every session read the decision record's D8 names is routed through the filter, and that the tests asserting the exact `statusFormatSessions` output or other `list-sessions` argvs are in the Edit list or provably unaffected, since the card claims every other test is unaffected."
  - "Card order and the shared symbols: cards 3 and 4 both edit `ensureServerAndSessionLocked`, and card 4 edits the `plan:` `TestBoot_TmuxConf` that card 3 creates. Settle that each `Uses` and edit target resolves to an earlier card, and that card 3's `reedSessionOption` and `tmuxConfClaimedOption` are consumed as card 4 assumes."
  - "Granularity and sizing of card 3: it creates nine symbols and edits nine targets, spanning spawn argv, identity read, guard argv, guard runner, two test helpers and two new test files. Settle whether it is one independently reviewable unit or whether the test-helper refactor (`newHubEngine`, `bootOwnServer`) or the identity read belongs elsewhere, without violating the glyph and ordering checks."
  - "Test economy: settle which existing tests already cover the planned `TestTmuxConfPath` refusal rows versus the three new `TestUp_BootValidation` rows, and `TestTmuxConfBootGuard`'s warn-on-error and read-back rows versus `TestBoot_TmuxConf`'s parse-error, run-time-error and untargeted-`destroy-unattached` subtests. A finding must name the covering test."
  - "Re-sign arrow and member glyphs: card 2 changes `preflightBootHost`'s results and edits its two callers, and card 3 edits `ensureServerAndSessionLocked` and `New`. Settle for each `Edit` glyph whether the `Intent` changes parameters, results or receiver, and whether the arrow is present or absent accordingly."
  - "Attack-surface statements: settle that the `Intent` of each card introducing a new key, bypass or guard change (card 1 key, card 3 non-fatal guard exit status and closed pin set, card 4 filter hiding sessions) names what it lets through and what bounds it."
  - "Verify coverage: settle that `## verify:` runs the tests of every package a card targets (`internal/reedengine`, `internal/reedengine_test`, `internal/configreg_test`, `internal/testkit/tmuxkit`, `cmd/lyx` budget data), that the `tmux` tier is compiled only, and that the section follows the template's order. Check the card-level `**Verify:**` lines on cards 3 and 4 against what the spec permits."
---

Entries name where to look; none states a verdict.
`support-log.md` was not read.
