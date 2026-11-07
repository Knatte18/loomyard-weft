```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens review of the `beamHub` → `beamTemplate` diff in `internal/battencli/beamguard_integration_test.go`.

**Per-change set list**

1. `beamHub` now calls `hubforge.CopyRegistered(&beamTemplate)` instead of `CopyHub` plus three `AddPairWith` calls.
   - Callers: `TestResolveBattenPrime` (line 61), `TestVerbs_RunFromABeam` (166) and `TestCancelCmd_RemoveRefusesFromInsideTheBeam` (237), all in the same file. The only other `beamHub` hit in the package is its definition. Covered.
   - The `h.Location` returned by `CopyRegistered` is what `PrimeName`, `PairCodeWorktree`, `PrimeRecords` and `BoardDir` already used from `CopyHub`. I found no signature difference in the context I hold. Covered.

2. The new key `"battencli-beam"` must be unique among the binary's registrations and shape names.
   - A grep across the repo for `battencli-beam` and `CopyRegistered` outside `hubforge` finds only this registration. The shape names have the form `.+...`, so they cannot collide. Covered.

3. The replay-ordering swap in `TestResolveBattenPrime` and `TestVerbs_RunFromABeam`.
   - Both now take the copy before `boardkit.InstallReplay`. The third caller (237) installs no replay. Covered.
   - Other pair-building fixtures in the package are `noticeHub` (`notices_integration_test.go:27-29`), `beamfork_integration_test.go:88`, `beamland_integration_test.go:42-44` and `startguard_integration_test.go:36`.
   - `noticeHub` repeats the same three pair adds, but the decision record says `notices_integration_test.go` is left untouched, and it records a shortname first. The diff leaves it alone, so there is no finding.
   - The other three are single sites with different slugs and options, and D7 only moves start states that at least three sites build. Not in scope.

4. Documentation and rules.
   - The `beamHub` doc comment was updated. `beamTemplate` has a doc comment. The file header still describes the file's content correctly.
   - `PATTERN-test-isolation` says "no test package wraps `hubforge` in its own fixture type". `beamTemplate` is a `hubforge.Registration` value and `beamHub` is a plain helper returning `*hubforge.Hub`, so it complies.
   - The decision record says the `PATTERN-test-isolation` background gains a registered-template paragraph. The pattern background I read does not show it in the portion I viewed. That belongs to the hubforge task, not to this diff, and the diff touches no `PATTERN.md` or `pattern/` file. No declared PATTERN change is attributed to this part, so there is no finding.

5. The build's `testing.TB`.
   - The `Build` function takes `tb` and passes it to `AddPairWith`. It calls none of `Setenv`, `MkdirTemp` or global git config, and keeps no `Context()`. Covered.
   - `AddPairWith` takes `SkipPush` from the options, so the build pushes nothing. `requireBoardAtUpstreamTip` therefore compares the board's HEAD with the bare's tip; I did not verify that `Add` with `SkipPush` leaves the board at the tip. The old fixture used the same options and the build is only the old steps moved. The template-side check covers it at test time. No finding.

6. Commit message and departures.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.
   - Its claims hold against the diff: the steps are moved with the same options and order, and the copy comes before the replay in two tests. The `summary.md` claims hold as well.

**Findings:** none.

**Tool calls:** none. The manifest and the earlier greps already held every definition and call site this lens needed.
