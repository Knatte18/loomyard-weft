```yaml
lens: code-affected-set
outcome: reported
```

The task was the code-affected-set lens: everything outside the changed lines that the diff's changes force. It found no findings.

**Per change, the set found:**

1. **`TestGateRedFirst_Dotnet` becomes a table whose rows run in parallel, each in its own `hubforge.CopyHub` hub.**
   - Covered: `pattern/PATTERN-test-speed.md`'s "Tests run in parallel". The diff removes the old comment that justified running serially, which is now stale, and leaves `t.Parallel` on the test and every row.
   - Covered: `pattern/PATTERN-test-isolation.md`'s "Hub fixtures". Each row takes a `*hubforge.Hub` directly and wraps it in no fixture type of its own.
   - Covered: the gate-slot pool. `internal/gateslot/pool.go:118` keys it per hub `boardDir`, so a hub per row gives each row its own pool. The fixture's `slots: 1` is per hub.
   - Covered: the red-first checkout. `lymtemp.MkdirTemp("red-first-")` (`internal/gatecli/redfirstdotnet.go:90`) gives every call its own directory.
2. **The doc comment says "Each call builds the solution cold in a fresh checkout of the base".**
   - Covered: `redfirstdotnet.go:123` sets `run.checkoutRoot = checkout`, and `run.go:101-102` points the build and tier dirs at that checkout. A fresh export per call means no `obj/` carries over, so the comment holds.
   - The comment still holds `//lym:only upkeep`, as PATTERN-test-speed's "every test that builds a real .NET project" rule asks.
   - Its reference to `TestGateTest_DotnetFixture` still matches `dotnetfixture_integration_test.go:36-39`.
3. **The `ATestThatPassesOnTheBaseExitsWithTheGreenCode` row is dropped and the `gateslot` import goes with it.**
   - Covered: no doc, stencil, test-redundancy report or "Kept" list names the row.
   - The rows of the same name in `redfirst_integration_test.go:134,141` (the Go scoper) and `redfirstdotnet_test.go:173,177` (stand-in tiers) belong to other tests, and the diff leaves them untouched.
   - The claim in the commit message and summary that `TestDotnetRedFirstCallRun_…` still pins the green exit holds: `redfirstdotnet_test.go:177-178` asserts both `RedFirstGreenExit` and "does not show the bug".
   - The claim that `TestGateTest_DotnetFixture` pins real vstest counts holds: `dotnetfixture_integration_test.go:68`.
   - Whether those stand in "no looser" for the dropped real-dotnet assertion belongs to the tests and Loosened-Check judgement, not this lens.
4. **Renaming `ATestThatFailsOnTheBaseExitsZeroNamingIt` to `OnlyTheTestThatFailsOnTheBaseIsNamedRed`.**
   - Covered: no doc or fixture names the old dotnet-integration row.
5. **Docs naming `TestGateRedFirst_Dotnet` elsewhere.**
   - `docs/research/speed/test-time-measure-2026-11-27/report.md:89,167` and `calls.tsv:236,302` quote times from one past run. They are a measurement report, which the "Don't pin a perishable specific" exception exempts, so they are no finding.
6. **Rubric items.**
   - Stencil wording: the diff changes no stencil.
   - Declared PATTERN changes: the directive declares none.
   - Commit messages and departures: there is no `Pattern-Departure:` trailer. The commit message's claims are borne out by the files cited above.

**Findings:** none.

**Tool calls:**
- A grep for the test name and the old row names across the repository. The manifest lacked docs and fixtures that name them.
- A grep for `checkoutRoot` in production `gatecli`. The manifest lacked where the checkout root redirects the build, which the doc comment's "cold in a fresh checkout" claim rests on.
