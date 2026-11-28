```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions.** The answer key is the decision record alone, with no `Added after Discussion` entries.
- G1, from `## Idea`: find where `TestGateRedFirst_Dotnet` spends its time and cut it, for example by sharing one built .NET fixture. That source is `_lym/discussion/decision-record.md:19`.
- `## Seen` and `## Which runs pay for it` (`decision-record.md:4-15`) are background with no ask in them, so neither is a decision.

**Per change:**
- C1: each row copies its own hubforge hub, commits the fixture there and runs with `t.Parallel()`. Evidence: `001-internal-gatecli.diff:107-118`. This traces to G1 (cut the time).
- C2: the `ATestThatPassesOnTheBaseExitsWithTheGreenCode` row is dropped. Its `RadiusIsOne` method moves into the red row, now named `OnlyTheTestThatFailsOnTheBaseIsNamedRed`. Evidence: `diff:92-99`, `diff:136-143`. This traces to G1: one fewer cold build. The `Loosened-Check:` trailer on f8de6d0f7 explains the loosening, which other lenses judge.
- C3: the closures `writeCircle`, `writeTest` and `run` become a table and one loop, and the `path/filepath` and `gateslot` imports go. Evidence: `diff:5-15`, `diff:58-82`, `diff:144-167`. C1 and C2 force this.
- C4: the doc comment is updated ("adds tests of its own", plus three new lines on where the time goes and why rows run in parallel), and the serial-run comment is removed. Evidence: `diff:23-36`. C1 and C2 force this.
- C5: commit bcbce6494 re-breaks the C4 comment at a sentence boundary. C4 forces this.

**Per decision:**
- G1: built by C1 and C2. C3, C4 and C5 follow from them. No change shares a built fixture, but the entry offers that only as an example ("for example"), so leaving it out is no omission.

No change traces to nothing, and no decision is left unbuilt.

**Findings:**

1. LOW, class `scope`, location `internal/gatecli/redfirstdotnet_integration_test.go:27` (the new doc-comment line "Each call builds the solution cold in a fresh checkout of the base, which is most of a row's time").
   G1's "find where it spends its time" part rests on a breakdown the diff never measured.
   The only figures are whole-row totals in `_lym/landing/summary.md`, `## Measurements` table ("12.37 s, 7.34 s, 10.58 s in series").
   No figure splits restore, build and test run within a row.
   So the claim that the cold build is "most of a row's time" is asserted, not found.
   Fix: give a per-phase timing of one row in the summary, or soften the comment to what was measured.

Tool calls: none; my inherited context held the decision record, the diff, the commit log and the summary.
