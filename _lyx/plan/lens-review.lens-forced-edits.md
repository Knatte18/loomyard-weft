```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of the 12-card dotnet-scoped-verify plan. I found 5 defects.

1. **MEDIUM — Cards 2 and 11, build timeout way forward.** The Merge-In and Finalize stuck reason for a timed-out verify still tells the agent to rewrite a tier, not the build.
   - Card 2 makes `FindTierTimeout` return `build` for the new build timeout line, so `verifytree` marks a build timeout as `TimedOut`.
   - But `internal/landingshed/verifygate.go:271` always names `lyx config gate --set 'tiers.tier<N>={...}'` and "a tier that is slow but healthy". For a build timeout that names the wrong key, against `PATTERN-refusal-way-forward`.
   - `internal/landingshed/verifygate_test.go:261` asserts that text.
   - No card lists the message's enclosing member or that test.

2. **MEDIUM — Card 2, Decision 6: nothing reads the record's `build` flag.** `LandingFailure.Build` is written, but no reader or prompt uses it.
   - Decision 6 says `build: true` exists "so the landing fixer is pointed at a compile break, not at tests".
   - The fixer prompt `contracts/stencils/landing/landing-template-fixer.md:8,28,31` only says "fix the merge or the failing tests" / "fix the failing tests only", and only names the record's path (line 22).
   - No card adds a line for a build failure. As planned, the flag points no one anywhere.

3. **LOW — Cards 1, 2, 8, 9, 10: `**Extends:**` used on docs whose existing sentences get rewritten.** Per the rubric, these refs belong under `**Edit:**`.
   - `internal/gateslot/doc.go:66` ("a scoper other than `go` or `none`") becomes wrong under card 1.
   - `internal/gateslot/doc.go:111` (records named `<stamp>-tier<N>-<pid>.json`) and `:116` (`FailedTier` finds the first failing tier line) become wrong under card 2.
   - `internal/landingshed/doc.go:123` ("The tier is the first failing tier the log's … summary lines name") becomes wrong under card 2.
   - `internal/gatecli/doc.go:55` ("Under the `none` scoper each of the three forms is whole") becomes wrong under card 10.
   - `internal/gatecli/doc.go:125` (what `tierFingerprint` hashes) becomes wrong under card 9.
   - `internal/gatecli/doc.go:59` (an empty scope "runs no listed tier's command … takes no slot") becomes wrong under cards 9 and 11.

4. **LOW — Card 11: the `lyx gate report` help text goes stale.** `internal/gatecli/report.go:34-35` (`reportCmd`'s Long text) lists the row fields as tier runs, carried, wait, wall and launch. Card 11 adds `build_runs` and `build_seconds` but doesn't list `internal/gatecli#reportCmd`.

5. **LOW — Card 11: stand-in comments go stale.** `runStandIn` gains a `sharedbuild` build role. But the `standInRecord.Build` comment says it "is set for the `go build` of the prebuild" (`internal/gatecli/test_integration_test.go:75`), and the `helperRoleStandIn` comment (`:44-45`) describes only the prebuild's build. `internal/gatecli#standInRecord` and the const block aren't listed.

Two things outside this lens, for the coverage lens:
- The `--card` refusal loop at `internal/gatecli/scopes_integration_test.go:289` covers only `go` and `none`, and card 8 adds no `dotnet` row.
- Card 1 rewords the scoper refusal. Its one exact-text assertion, `internal/gateslot/config_test.go:148`, is inside the listed `TestLoadConfig`, so that's covered.

Tool calls:
1. Searched for the old scoper refusal text, `ConfigOpenMaps` assertions, template goldens, readers of the `-tier<N>-` file stem, summary-line parsers and `plankit.Repo`. The plan changes these strings and helpers, and the exploration hadn't searched their readers.
2. Read the sandbox-coverage, gitrepo-boundary, spawn-observability and leaf-package allowlists, to check whether `internal/dotnetimpact` must be registered in any. None needs it.
3. Searched `contracts/`, `docs/` and `pattern/` for scoper values and landing-failure-record readers, and production code for the `tiers.tier<N>` way forward. The exploration hadn't read the fixer stencil body or the landing timeout message.
4. Searched for tests of `FindTierTimeout` and `TierTimeoutLine` and assertions of the landing timeout message, to confirm the test glyphs card 2 lists and find the unlisted `verifygate_test.go:261`.
