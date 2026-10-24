```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward — whether each refusal, stop and recovery step the cards add leads to a next step that can succeed.

**Findings**

1. MEDIUM — Card 2 / Decision 6 (build timeout at Merge-In and Finalize): when the build times out, the landing Stuck reason sends the operator to rewrite a tier, which cannot fix the build.
   Card 2 makes `FindTierTimeout` return `build`, so `verifytree` marks the result `TimedOut` with a Detail naming `gate build exceeded timeout_min …`.
   But `internal/landingshed/verifygate.go:270-271` words every timed-out verify the same way: "rewrite the tier whole from the prime with `lyx config gate --set 'tiers.tier<N>={...}'`".
   Raising a tier's `timeout_min` does not raise `build.timeout_min`, so following that step reproduces the same timeout.
   No card edits this message, though Decision 6 says the build timeout line names `lyx config gate --set 'build={...}'`.
   Proposed fix: card 2 adds the `timed out` message in `internal/landingshed/verifygate.go` to its Edit list (the glyph of the enclosing function, from quarry). That message names `build={...}` when `result.Detail` names the build, and a row of `TestVerifyGate_CheckMapsVerifyResult` reaches it.

2. MEDIUM — Card 2 / Decision 6 (failed build at Finalize): the landing fixer spawned on a `build: true` record is told to fix only failing tests, and that record holds none.
   `contracts/stencils/landing/landing-template-fixer.md:33` says "In `finalize` mode fix the failing tests only".
   Card 2 writes the record with `Build` true and empty `Tests`, meant to point the fixer at a compile break, but no card changes the fixer's prompt.
   In finalize mode the fixer therefore has no failing tests to fix, and the stencil forbids anything else, so the compile break can stay unfixed until the attempts run out.
   Merge-In mode says "from the failing tests" as well (line 29).
   Proposed fix: card 2 extends `contracts/stencils/landing/landing-template-fixer.md` so a record with `build: true` means: fix the compile error the log names. Under `PATTERN-stencil-ownership` that is a pointer to the record's field, not a restated format.

3. LOW — Card 8 (`--guards` under `dotnet`): the reused refusal's way forward offers a step that is wrong for a .NET hub.
   `internal/gatecli/tiers.go:118` says "drop --guards, or set scoper to go".
   Under `scoper: dotnet`, switching to `go` makes every C# tier run whole, or breaks the derivation since there is no `go.mod` (`impactset.DeriveTargets` falls back to whole).
   Only "drop --guards" progresses.
   Card 8 reuses the message unchanged and plans a `TestPlanTierRuns` row that asserts only that the refusal names the scoper.
   Proposed fix: card 8 words the refusal so that under a non-`go` scoper it offers only dropping `--guards`.

The other refusals and stops progress from the state they leave:
- card 1's load refusals, each naming `lyx config gate`, with `build` an open map writable whole;
- card 2's build timeout line naming `build={...}`;
- card 6's and card 8's cancelled-context error, which re-runs the same command;
- card 11's blocked tiers naming the build log, re-run once the code is fixed, with no green entry written;
- card 11's slot-busy wait for a build-only tier, which reuses `acquireSlot`'s run-again answer;
- card 12's skips, each naming its reason.

Tool calls:
- Read `internal/landingshed/verifygate.go:262-276` and the webster verify-gate and report messages, to see the timed-out wording each verify site prints; my context held only the first 140 lines of `verifygate.go`.
- Read `contracts/stencils/landing/landing-template-fixer.md:15-40`, to see what the fixer does with a landing failure record; my context had only the grep hit naming the record path.
