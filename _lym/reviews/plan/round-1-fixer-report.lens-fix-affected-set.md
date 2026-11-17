```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each set-premise finding (F3, F8, F10, F11) and checked that the fix covers every member.

**Findings I left unchecked:** none.

1. **BLOCKING** — card 8's Edit list and its "Tests, tier0" paragraph (the change made for F3).
   - Summary: card 8 changes `Deps.Owner` to return `(owner, elsewhere, err)`, but one beamcli test that calls it with the old two-value shape is still not listed, so beamcli's tests won't compile at card 8's gate.
   - Evidence: `internal/beamcli/dobbydeps_integration_test.go:392` reads `if owner, err := deps.Owner(); err != nil || owner != foreign {`. That line is inside `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`. Card 8 lists only dobbyengine tests and no `internal/beamcli#TestHubDobbyDeps_*`.
   - The other members of the set are covered:
     - In `overviewdobby_test.go:164` the `BeamPair` literal uses keyed fields, so it still compiles.
     - The `Machine:` fields in `beamcli/create_test.go` and `land_test.go` belong to other `Deps` types.
     - `handover_test.go:96` sits in `TestRun_ForeignOwner`, which card 21 rewrites.

2. **LOW** — card 23's new text-follow paragraph (the change made for F10).
   - Summary: once card 23 lands, landingshed's operator-facing texts still describe the owner pause, and no card lists them.
   - Evidence:
     - `internal/landingshed/finalize_owner.go:242` says "leaving the owner pause standing".
     - `internal/landingshed/parentguard.go:106` and `:128` name "an owner pause" as a cause of the lock release and say "`lym loom resume` once no owner pause stands".
     - `internal/landingshed/doc.go:321`, `:540` and `:543` carry the same wording.
   - A grep of cards 0–25 finds no landingshed target.
   - The remaining owner-pause readers are fine: pairteardown's followup, battencli's `owner.go` and `cancel.go`, and pauserecord's `OwnerEntryExempt` still read the lingering `OwnerCaller` entries the upgrade keeps declared. `tendercli/doc.go:64` is covered by card 24.

F8's set is covered: the only callers of `commitRunIntent` are the two closures at `runintents.go:63` and `:76`, both inside `hubRunIntents`, which card 19 now lists under Edit. F11's set is covered: `PATTERN-cli-cobra.md` has no other line that names machinecli's calls.

Tool calls:
1. grep for `BeamPair`/`StartDeps` literals, `.Owner()` calls and `Machine`/`PauseOwned`/`LiftOwnerPause` assigners across all test packages, because the parent's search had not covered beamcli's integration tests.
2. grep for callers of `commitRunIntent`, owner-pause texts across `internal`/`pattern`, and `pairteardown` in `PATTERN-cli-cobra`, to rerun the searches behind F8, F10 and F11.
3. awk and grep to name the function holding `dobbydeps_integration_test.go:392` and list the owner-pause phrases in the files found, since my context lacked both.
4. grep of the plan cards for landingshed, `tendercli/doc.go`, followup and `TestHubDobbyDeps` targets, to see whether any card covers the remaining texts.
