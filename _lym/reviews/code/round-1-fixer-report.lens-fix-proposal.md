```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F8).

Two LOW findings; the rest held up against the cases I built.

1. **LOW — F7, HubEnvDigest assertion.** The changed text is `stamp.TiersDigest == "" || stamp.HubEnvDigest == ""` in the first `TestGateTestMapLinkedPackages` subtest.
   - The fixture declares no hub env.yaml variables, so the digest input is `strings.Join(nil, "\n")`, the empty string.
   - Hashing an empty string still gives a non-empty digest, so this check cannot fail, whatever `environmentStamp` does with the hub env.
   - The proposal's "non-empty" check therefore pins nothing for `HubEnvDigest`.
   - Evidence: `testmappackage.go` `environmentStamp` sets `stamp.HubEnvDigest = testcover.ContentHash([]byte(strings.Join(r.hubEnv.Entries(), "\n")))`.
   - A real pin would set an env.yaml variable and assert the digest differs from the empty-env digest.
   - I did not run this, so the claim rests on `ContentHash` hashing the empty string to a non-empty value, as a hex digest would.

2. **LOW — F1, `--max` no longer bounds the cost of a dirty linked file.**
   - A before-build skip now gives its cap slot back, but `mapPackage` has already run `holdSlot` and `LinkedPackages` (a `go list`) by that point.
   - A dirty widely linked file therefore makes every stale package with mapped tests pay a `go list`, and a slot acquire on the first one, with no cap on the number.
   - This is the behaviour the review asked for, and `doc.go` and the `testMapCmd` help text now say so.
   - Neither says the skip is found only after the slot is held and `go list` has run, so a reader may expect it to cost nothing.

Cases I built that held, with no finding:
- **F1:** the sentinel-free bool return is safe. No other caller of `mapPackage` exists, and the build-failure and after-tests paths still count.
- **F2:** `TestDirtierLandsLinked` only acts when `testMapLandEnv` is set, so the earlier subtests that run `dirtier` are unaffected.
- **F4:** the new fixture directories do not change `modulePackages`, because the walk skips them or they hold no direct Go file.
- **F5:** with `scratch.txt` absent at call start, `follower` skips with the before-build cause only if its status read is fresh, so the subtest distinguishes a stale read.
- **F8:** the doc claim "only misses files, never adds a false one" holds. Cutting a directory out of the owner removes hashes and adds none.

Tool calls: none. My context already held every changed file.
