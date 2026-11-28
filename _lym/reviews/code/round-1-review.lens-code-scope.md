```yaml
lens: code-scope
outcome: reported
```

I traced the diff's changes to the board entry's goal points, under the code-scope lens.

**Decisions** (the record's goal points under `## Goal`; the record has no `Added after Discussion` entries):
- G1: the stalemate check hashes the same review target the reviewers read, so a round whose fixer changed the decision record is never a stalemate.
- G2: an empty hash input fails loudly and never counts as an unchanged target.

**Per change:**
- C1, commit 4a23f3a41, traces to G1. `internal/shedrecipe/entries_burler.go:59` moves `reviewTargetSnapshot`'s `WorktreeRoot` from `env.WorktreeRoot` to `env.AnchorPath`, and lines 53-54 add a doc comment. The test edit in `internal/shedrecipe/entries_burlerhalves_test.go:245-255` is forced by C1.
- C2, commit f635cccab, traces to G2. `internal/shedadapters/targetsnapshot.go:114-116` makes `overlayID` return an error on an empty file list, and line 45 adds a doc line. The new `TestTargetSnapshot` subtest at `internal/shedadapters/targetsnapshot_test.go:87-97` is forced by C2.
- C3, commit 060437e3c, is forced by C1: it rewraps C1's new doc comment and changes no behaviour.

**Per decision:**
- G1 is built by C1. The review row (`entries_burler.go:182`) and the Bouncer (`entries_bouncer.go:312`) both build their snapshot through `reviewTargetSnapshot`. The root now matches `hubgeom.burlerGeometry`'s `WorktreeRoot: l.AnchorPath()` (`internal/hubgeom/hubgeom.go:169`), the geometry `loomcli` wires (`internal/loomcli/wiring.go:718`).
- G2 is built by C2. A failed id read leaves `TargetID` or `PostFixTargetID` empty (`burlerreview.go:230-232`, `bouncerstalemate.go:34-37`), and `roundUnchanged` counts a round with an empty `TargetID` as changed (`bouncerstalemate.go:55`).

No change traces to no decision.

**Findings:**

1. LOW, class `scope`.
   - Location: `internal/shedadapters/targetsnapshot.go:114-116`, with the unchanged callers at `burlerreview.go:230-232` and `bouncerstalemate.go:34-37`.
   - Summary: G2's "fail loudly" is only partly built. The id read now returns an error, but both callers only log it at Warn and the round goes on, so a missing target shows only in the log.
   - Evidence: the board entry's `## Expected` line 2 (`_lym/discussion/decision-record.md:16`) against the callers' `logger.Warn` handling. The change description's "Design decisions" (`_lym/landing/summary.md:22`) states this choice. The "never counts as unchanged" half is met, so this is only a question of how loud "loudly" must be.

I made no tool calls; my inherited context held everything this lens needed.
