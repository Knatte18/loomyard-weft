# Fixer report — plan, round 1

Every finding in `round-1-review.md` is fixed.
`lyx loom validate-plan` passes on the edited plan (`ok: true`, informational `create-new-unit` findings only).

## Changes

- **F1 (BLOCKING), card 31:** creates `internal/gitkit/testmain_test.go` as a `!integration` sibling.
  `internal/gitrepo/testmain_test.go` drops its `//go:build integration` line, since the sibling name is taken, so its one `TestMain` compiles under every tag set.
  The Intent names both packages and the doc-comment rewrite.
- **F2, card 12:** the deletion list now names `shedcli`'s `TestRunCLIIn_BareListingNeedsNoGitRepository` and `loomcli`'s `TestCommand_ReviewSubtree_ExactChildren`.
  It also names `boardcli`'s `TestCLIUnknownSubcommand`, whose name falls outside the enumerated pattern.
- **F3, cards 21–22:** card 22 adds `internal/loomcli/smoke_parentreview_test.go` and `internal/shedadapters/bouncer_fixture_test.go`, covering `newBouncerFixture`'s default map and `writeBouncerStencils`.
  It widens its grep to name-to-content maps and gains `**Verify:** go test -tags smoke ./internal/loomcli/`.
  Card 21's smoke sentence is corrected and points at card 22.
- **F4 and F13, card 28:** the core takes an optional after-commit callback, which `battencli` uses for its marker write.
  The Intent states what moving `CommitStatusMessage` lets through.
  For the bound, `loomcli`'s prefix stays pinned by the protected integration test, and one of `battencli`'s kept marker-skip tests asserts the `batten: <producer> -> <state>` message.
- **F5, card 15:** adds `internal/loomrecipe/interruptpolicy_meta_test.go` and has it, and `recipe_test.go`, read the engine mapping from the one table.
- **F6, cards 23–25:** a claim may name a section heading that scopes its check.
  Card 24 keeps the section-scoped `websterengine` checks scoped, and card 25 does the same.
- **F7, card 25:** drops `loomengine/anomalybody_test.go` and `shedadapters/rubric_test.go` from the targets and says why.
  The open "another test file" clause becomes report-not-move.
- **F8, card 18:** drops `cliwire/cliwire_test.go`, `webstercli/wiring_test.go` and `websterengine/template_test.go` from the targets.
  It states that the two placeholder writers are not plan writers, and its ImpactSummary is updated to match.
- **F9, cards 31–33:** each records that HEAD has no TMUX-reliant test or environment-from-scratch spawn, and turns the open fix into a check whose hit is reported as a finding.
  Card 33's ImpactSummary drops "plus any environment-from-scratch spawn found".
- **F10, cards 24, 25, 27:** each Intent now states what the removal lets through and what bounds it.
- **F11, cards 2, 17, 20, 30:** cards 17, 20 and 30 state the new kit's package doc.
  Card 2 restates `docs/overview.md`'s Testkit scope sentence.
- **F12, card 30:** `tmuxkit_test.go` asserts that `reedengine.ServerName` of an over-long hub path fits the 61-byte bound, and `internal/reedengine/server.go` joins the card's `Uses`.

## Deferred

Nothing deferred.
