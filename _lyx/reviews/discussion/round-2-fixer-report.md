# Fixer report — discussion round 2

## Changes

- F1, fixed as proposed: Decision 2's card seed adds a glyph's own name as a type when its owner chain is empty.
  Decision 3's `--card` bullet applies the diff path rules to a Target's file: an inert Target is dropped, and a non-`.cs`, non-inert file, a deleted file or a namespace-only self glyph runs whole.
  The card criterion covers both cases.
- F2, fixed as proposed: the `PATTERN-config-layers` Constraint puts `build` in `gateslot.ConfigOpenMaps` beside `tiers` and `tier_parallel`, with `gateslot.LoadConfig`'s Decision 6 refusals validating its keys.
- F3, fixed as proposed: Decision 4 gains a Bound on the tier `filter` at whole scope, the vacuous pass, the `tiers` key bound and the `docs/dotnet-hub.md` guidance.
- F4, fixed as proposed: Decision 2's lexer keeps the code inside interpolated-string holes, regular and raw, as identifier tokens.
- F5, fixed as proposed: Decision 2's result bullet spells the term as `FullyQualifiedName~` plus the namespace and class chain, a nested class joined by `+`; the test-class criterion adds a nested class.
- F6, fixed as proposed: Decision 6's failed-build bullet and the build criterion use the plan-order first-failed-run rule.
- F7, fixed as proposed: Decision 3's empty-diff sentence and the `inert_paths` criterion except a listed `carry: false` tier with `shared_build`.
- F8, fixed as proposed: Decision 3's empty-test-scope paragraph and its criterion state the per-tier rule.
- F9, fixed as proposed, then refined in the self-check: Decision 4's `{filter}` bullets are keyed on each invocation of the tier's own share.
- F10, fixed as proposed: Decision 2 skips `.scratch/`; Decision 3 falls back to whole for a selected test class with no `.csproj` ancestor up to the tier's `dir`, or with more than one in its nearest such directory.
- F11, fixed differently: the premise holds for the record, which never said how untracked files are read, but `impactset` already reads untracked files through `git ls-files --others --exclude-standard` (`internal/impactset/targets.go:102`), not `git status`.
  Decision 5's first bullet and Decision 2's diff seed count untracked files as changed, read as `impactset` reads them, instead of through `git status` as proposed, so one way of reading untracked files serves both scopers.
- F12, fixed as proposed: the first Open risks bullet says a closure across every test project runs one `dotnet test` per project in sequence, which can cost more than one solution-level run.
- F13, fixed as proposed: the tier1 criterion requires two projects MSBuild builds in parallel, built with `-m`.
- F14, fixed as proposed: support-log Q2 names `merge-in` and `finalize` as the whole gates, with `webster` at scope `diff` per Q12.
- F15, fixed as proposed: Decision 10 calls Merge-In the one planned whole-solution run, besides the gates Decision 3 sends whole.

## Self-check

- fix-way-forward, MEDIUM, a card whose Targets are all inert has no stated outcome: fixed as proposed, merged with fix-consistency 3 and fix-proposal 3.
  Decision 2's result bullet, Decision 3's empty-scope paragraph, Decision 6's no-build bullet and the card criterion now give it the empty scope, so no tier runs but a `carry: false` one, and no build runs.
- fix-consistency, MEDIUM, the per-tier empty-test-scope rule reads as one build per opted-in tier: fixed as proposed, merged with fix-proposal 2; Decision 3 and its criterion run the build once, inside the first opted-in tier's slots.
- fix-consistency, LOW, support-log line 51 still says the build is the first failing run: fixed as proposed.
- fix-consistency, LOW, Decision 2's result bullet misses the all-inert card: fixed as proposed, merged into the first line.
- fix-claims, LOW, a type glyph's name or owner component can carry a generic or arity suffix: fixed as proposed; Decision 2's card seed strips the suffix.
- fix-proposal, MEDIUM, F9's share-keyed bullets contradict the no-`{targets}` line and the over-budget fallback: fixed as proposed; Decision 4 and its criterion key the rendering on an invocation the `dotnet` scoper narrowed to a selected project with its terms, and both cases render the tier filter alone.
- fix-proposal, MEDIUM, F8's per-tier rule against one build per call: fixed, merged into the fix-consistency MEDIUM line.
- fix-proposal, MEDIUM, all-inert card: fixed, merged into the fix-way-forward line.
- fix-proposal, LOW, the forced MSBuild node outlives the tier1 test: fixed differently.
  The proposed teardown, `dotnet build-server shutdown`, ends every node of the user's SDK, other worktrees' gate runs included, and MSBuild offers no per-fixture shutdown.
  The criterion instead names the idle node as process-global state the test leaves until the SDK's idle timeout, as the testing rules allow for process-global state a test names.

## Disputed

None.

## Deferred

None.
