```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1, F2, F3, F5–F8, and F4's doc option), looking for a case where the proposal is wrong or breaks something.

**1. MEDIUM: F7's refusal names a way forward that cannot resolve the case.**
- Changed text: `decisionParent` in `internal/loomcli/decision.go`, `parent.Name == ""` branch ("the beam %s has no resolvable orch … start it with `lym beam start %s`"), and the tier-1 subtest `ABeamHubWithNoShortnameIsRefusedNamingTheWayForward`.
- Evidence: `hubgeom.NearestLiveOrch` (`nearestlive.go:26-31`) returns an empty `Parent{}` only when the hub has no recorded shortname.
- When the shortname exists but the beam's orch is not live, it walks up and returns the prime's orch with a non-empty `Name` (the "prime whatever its state" fallback).
- So the empty-`Name` case is always a missing shortname. Starting the beam's orch with `lym beam start` does not fix that, and the re-run is refused the same way.
- The message also says the beam "has no resolvable orch", which misdescribes the cause.
- The test pins this wording by removing only the shortname file and asserting `lym beam start the-beam`.
- Case where the proposal fails: a beam in a hub whose shortname file was lost. The operator follows the refusal, starts the orch, and is refused again.
- A correct refusal would name the missing hub shortname as the way forward.

**2. LOW: F2's test does not pin the claim it was added for.**
- Changed text: `TestWithProjectDirective` in `projectrecord_test.go`, "APairUnderNoBeamKeepsTheMarkerEmpty".
- The review's stated claim is that a pair under no beam gets a prompt byte-identical to one built without the key.
- The test checks only the `Values` map: the marker is `""` and the other values are unchanged.
- It does not check that a blank value renders the same as an absent key. That depends on the marker being in each table's `Optional`.
- `TestDiscussionTable` already asserts `Optional` for `DiscussionTable`. I found no assertion for `DarnTable` (`darnengine/spec.go:112`).
- A change that drops `project_directive` from the darn table's `Optional` would make every non-beam darn spawn fail validation, and this test would still pass.

**3. LOW: F1's test misses the "absent pair worktree is skipped" case.**
- Changed text: `TestMemberPairs_ReturnsEveryLocalPairUnderTheBeam`, and `memberPairs` in `projectrecord.go:145`.
- The `member-no-pair` member is dropped earlier, by `HasPair` in `ReadBeamLandBlockers` (`beamland.go`, `pairPresent`).
- The `os.Stat(...ErrNotExist) → continue` branch in `memberPairs` is therefore never reached.
- Deleting that branch leaves the test green, so the review's "skips an absent pair worktree" is not pinned.

No other finding. F4 (doc-only), F5, F6 and F8 hold up: the commit test's `CommittedAnchoredFile(record.rel)` still pins the rel/path pairing that F8 removed from the path test.

Tool calls made: none. Everything I used was already in my context.
