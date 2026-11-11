```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I list the test row that would fail if it broke, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| Replace set selects after the write's pull, takes open beams and the prime, ignores done entries, other owners and an empty name | `TestSetOwners/ReplaceSetSelectsAfterThePull` (`internal/boardengine/ownerset_integration_test.go`). Its `""` in `Replace` is pinned by `beamer` keeping `""`. |
| Replace set with no matching owner writes nothing | `Outcome_ReplaceSetWithNoMatchingOwnerWritesNothing` |
| Commit subject `board: owner X replacing …` | `ReplaceSetSelectsAfterThePull` (subject assertion) |
| Replace combined with Slugs, Prime, Claim or Takeover is refused (`ownerset.go:97`) | **none** |
| Replace under `BOARD_SKIP_GIT` / `BOARD_SKIP_PUSH` (`CommitWritten` path) | none. No LOW finding: the selection code is shared, so only the path differs. |
| `FromThisHost` | `TestFromThisHost` |
| Invalid name refused, nothing written | `TestName_Set/InvalidNameIsRefusedAndWritesNothing` |
| No directory or relative variable refused, naming the variables | `NoDirectoryIsRefusedNamingTheVariables`, `RelativeVariableIsRefused` |
| Unreadable file refused with its path | `UnreadableFileIsRefusedNamingItsPath` |
| Registry read failure refuses before any write | `RegistryReadFailureIsRefusedBeforeAnyWrite` |
| Unreadable board refuses before any write | `UnreadableBoardIsRefusedBeforeAnyWrite` |
| Collision with an entry owner refused, naming hub, slug and name | `CollisionIsRefusedNamingHubAndOwner` |
| Collision with an origin-only owner refused (pull happens before the read) | `TestName_SweepsHubs/CollisionWithAnOriginOnlyOwnerIsRefused` |
| Collision with the prime owner refused (`set.go:171`) | **none**. `HostnameNamePassesTheCollisionCheck` covers only the own-name pass. |
| Own former name, hostname name and a hand-broken file's last valid name pass the collision check | `OwnFormerNamePassesTheCollisionCheck`, `HostnameNamePassesTheCollisionCheck` |
| Hostname name taken as the old name; no hostname adds none | `NoFileTakesTheHostnameNameAsOld`, `NoHostnameAddsNone` |
| New name moved to the end of `names` | `SetNameOutOfOrderMovesToTheEnd` |
| Already-set re-run writes nothing but still sweeps with the same replace set | `AlreadySetWritesNothing`, and the re-run in `FailedHubIsReportedAndOthersSweep` |
| Invalid name dropped from a decoded file, `names` kept | `OwnFormerNamePassesTheCollisionCheck` (`wantInvalidDropped`) |
| Undecodable file replaced, names lost and reported | `UndecodableFileLosesItsNamesAndSaysSo` |
| Hand-written file with only a name sweeps the hostname name | `HandWrittenNameSweepsTheHostnameName` |
| Empty entries dropped and counted in the output | `EmptyEntryDropIsReported` |
| Failure of `state.WriteJSON` refuses with its way forward (`set.go:249`) | **none** |
| Pull skip or error is checked locally and named | `PullSkipOrErrorIsCheckedLocally` |
| Failed unpushed read is noted and the sweep continues | `UnpushedReadErrorIsNotedAndSwept` |
| Failed hub reported, others go on, non-zero exit, re-run sweeps it | `FailedHubIsReportedAndOthersSweep` |
| Lost entries reported per entry with owner and way forward, non-zero exit | `LostEntryIsReportedPerEntry` |
| Pushed-only re-run reports `pushed` | `UnpushedBeforeReportsPushed` |
| Nothing recorded reports `nothing to sweep` | `NothingRecordedReportsNothingToSweep` |
| Working-directory hub swept once, last when outside the record, container directory counted, plain directory adds none | `CwdHubInTheRegistryIsSweptOnce`, `CwdHubOutsideTheRegistryIsSweptLast`, `CwdContainerDirectoryIsSwept`, `PlainCwdAddsNoHub` |
| Real sweep: open and origin-only beams and the prime moved; done entry and `far` / `neve` owners untouched; working-directory subdirectory resolves to its hub | `TestName_SweepsHubs/RenameSweepsEveryHub` |
| `liveHubs` error wrapping (`hubs.go:22`) | none, but the seam fake covers the refusal. Trivial. |
| Show: hostname fallback (no file, no directory), file name with former names, empty entry skipped, writes nothing | `TestName_Show` rows `NoFileReadsTheHostnameRule`, `NoDirectoryReadsTheHostnameRule`, `FileNameWithFormerNames` |
| Show: bad file prints its error and path with a way forward and no name; relative variable and hostname error shown | `TestName_Show` rows `UndecodableFileShowsItsErrorAndPath`, `InvalidNameShowsItsErrorAndPath`, `UnreadableFileShowsItsPathAndRemoval`, `RelativeVariableShowsItsError`, `UnreadableHostnameShowsItsError` |
| Output names the recoveries (`recoveries` text) | `requireRecoveries` asserts only that the list is non-empty. The wording is static prose, so exempt. |
| `cmd/lym` wiring and sandbox exclusion | `seamsignature_test.go` (compile-time); the exclusion is covered by the sandbox coverage test. No separate finding. |

**Findings**

1. LOW, class `design`, `internal/boardengine/ownerset.go:97`, with the tests in `ownerset_integration_test.go`.
   - Summary: the refusal of a replace set combined with Slugs, Prime, Claim or Takeover has no test row.
   - Evidence: the diff's only `Replace` cases are the outcome row and `ReplaceSetSelectsAfterThePull`, and both set only `Machine` and `Replace`.
   - Fix: add one `TestSetOwners` outcome row per combination, asserting the refusal message and that nothing is written.

2. LOW, class `design`, `internal/machinecli/set.go:171`.
   - Summary: the prime-owner collision branch of `checkHub` has no test, so removing it would pass every test.
   - Evidence: `collidingOwners` and the integration collision case both use entry owners. The prime appears only in the passing `HostnameNamePassesTheCollisionCheck`.
   - Fix: add a `TestName_Set` row whose fake owners hold `Prime: "osl46"`. It should expect a refusal naming `OwnerWritePrime`.

3. LOW, class `design`, `internal/machinecli/set.go:249`.
   - Summary: the refusal on a failed `state.WriteJSON` of the machine file has no test.
   - Evidence: no row makes the write fail. `dirAtPath` fails earlier, at read time.
   - Fix: add a row whose file path sits under a read-only or non-directory parent, resolving a directory but failing the write. Assert the refusal names the path and the re-run, and that no owner write was sent.

**Tool calls:** none. The manifest already held every file this lens needed.
