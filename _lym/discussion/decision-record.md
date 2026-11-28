## Goal

```markdown
## Seen

The landed split of the long gatecli tier1 scenario tests made `TestGateTest_Scopes` and `TestGateTest_Scenario` parallel; their longest rows now take 2.9 to 7.3 s.
The package time did not fall: `TestGateRedFirst_Dotnet` took 31.0 and 38.9 s before and 47.5 and 44.4 s after (the after runs ran at a load of 31 and 46),
and `TestGateTest_DotnetFixture` takes about 21 s. The figures are in the landing commit's Measurements section.

## Which runs pay for it

Both .NET tests are marked `//lym:only upkeep`.
With `gate.scoper: go`, as this hub sets it, a `--gate` call of any other key skips them.
The landing commit's measurement command was a `--tier 1` call with no `--gate`, which runs them,
so the times above do not show the cost of a task gate.

## Idea

Find where `TestGateRedFirst_Dotnet` spends its time (dotnet restore and build, likely) and cut it, for example by sharing one built .NET fixture across its rows or with `TestGateTest_DotnetFixture`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - dotnet-redfirst-green-row-dropped (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
