## Goal

```markdown
## Seen

Code-Review round 1 of the task that restores the PR-Review agent session (#282) raised a BLOCKING PATTERN finding: the session tests and `envkit` stand in for repository modules.
The fixer removed `fakePRSessions` and envkit's `noopSessions` and runs the real `shuttleengine.Runner` over `shuttlefake`, adding `shuttlefake.ResumeEngine`.
Still hand-written: the `SessionLive` closures over the kit's strand table and in `envkit`, the `RegisterSession` closures, and the `MailAgent` closures, one of them returning a made-up "no such strand" error.
No kit holds a recorded session stand-in, and a real reed needs tmux while these tests are tier0.
The parent accepted the round with this open, to land the urgent fix.

## Idea

Give testkit a recorded reed and creel session stand-in usable at tier0, and move the PR-Review session tests onto it.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): Scope: recorded tmux stand-in under reedengine; creel and envkit later

Decision: Build option B: export a tmux-exec and pane-age seam on reedengine.New, add internal/testkit/tmuxrec with recorded tmux transcripts and their source lines, a tier0 replay test and a tier2 recorder and contract test, move sessionLiveness and sessionRegistrar from loomcli into reedengine, and run the PR-Review session tests over real reed and the real runner at tier0. Leave the MailAgent closures and envkit unchanged; they are filed as a board note for after push-retry-not-halt.

Rationale: It delivers the kit the entry asks for within PATTERN-test-stand-ins, which forbids a stand-in for reed or creel themselves. envkit.go is changed by the running push-retry-not-halt, and a real creel path needs a daemon, so both wait for a follow-up.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - registration-check-loosened (class consistency, severity MEDIUM, open in rounds 1)
  - replay-depends-on-host-path-and-os (class design, severity MEDIUM, open in rounds 1)
  - replay-ignores-call-stdin (class design, severity MEDIUM, open in rounds 1)
  - replay-refusals-untested (class design, severity MEDIUM, open in rounds 1)
  - replay-stale-read-after-mutation (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
