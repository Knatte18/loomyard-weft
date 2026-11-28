```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**Per change, the affected set I found**

1. **`gitexec.IsTransportFailure`: a missing local repository path now counts as a transport failure** (`internal/gitexec/transport.go:79-124`)
   - Callers:
     - `fabricengine/originreconcile.go:236,260,262,625`: covered by the reconcile row change.
     - `pushclassified.go:214` (`retryTransportPush`), `:367` and `:647` (classification), `:760` (`checkOrigin`), `:807` and `:813` (`awaitOrigin`): covered only for the `:647` record path, by the pushclasses row.
     - `executor.go:182` (`outcomeClassOf` → `OutcomeTransportFailure`): not covered.
     - `add.go:96` (`RetryOriginReadInTransit`): the fixture was changed to a non-repository directory, so the missing-path case is not covered.
     - `pull.go:639` (`withTransportCause`): not covered.
     - `pushanchored.go:459` (owed records push): not covered.
     - `shedtransient/shedtransient.go:34` (`TransientGitTransport`, which the shed driver waits out as a network failure): not covered, and this one is outside fabric.
   - Fixtures that still point origin at a missing path:
     - `fabriccli/adoptremote_integration_test.go:4,190,265` and `fabriccli/cleanupremove_integration_test.go:44-48,429` mean "unreachable" and stay consistent.
     - `fabricengine/recordsreads_integration_test.go:137`, `originreads_integration_test.go:87`, `pairparts_integration_test.go:118`, `pendingteardown_integration_test.go:80` and `destructivegaps_integration_test.go:150` (clone) assert only an error or a read with no network call, so they are unaffected.
   - Pattern docs: `pattern/PATTERN-pair-remote.md:29` ("owed push, on an unreachable origin") and `PATTERN-daemon-wakeups.md:103` are generic and still consistent. No declared PATTERN changes, so nothing to land.
2. **Package doc and `IsTransportFailure` doc comment** (`doc.go:23-24`, `transport.go:95-98`): set is the code itself; see finding 2.
3. **Test fixture moves from a missing path to an existing directory** (cleanupcli, `add_remoteleftover`, `originreconcile`, `pushanchored`, `pushclasses`): the comments at `add_remoteleftover_integration_test.go:10` and `pushanchored_integration_test.go:87` are updated, and `pushclasses_integration_test.go:31` stays accurate. Covered.
4. **Commit messages**: commit 6870ad49f carries a `Root-Cause:` trailer naming the defect the diff removes. No `Pattern-Departure:` or `Loosened-Check:` trailer is needed, since no assertion is loosened: the originreconcile row gains an assertion. Covered.

**Findings**

1. **MEDIUM**, `consistency`, `_lym/landing/summary.md:18-21` and `:47-48`.
   - **Summary:** "Every caller that classifies through this function follows" lists only the push classification and `ReconcileOrigin`. The blast radius `fabric` leaves out callers whose behaviour the change also flips:
     - `shedtransient.Class` (`internal/shedtransient/shedtransient.go:34`), outside fabric: a shed stop on a missing local origin now becomes a transient network wait instead of a stop.
     - `outcomeClassOf` (`internal/fabricengine/executor.go:182`).
     - `RetryOriginReadInTransit` (`add.go:96`), which now backs off and retries.
     - `withTransportCause` (`pull.go:639`).
     - The owed records push (`pushanchored.go:459`).
     - `checkOrigin` and `awaitOrigin` (`pushclassified.go:760,807,813`).
   - **Evidence:** the grep of `IsTransportFailure` callers; `shedtransient.go:30-35`.
2. **LOW**, `consistency`, `internal/gitexec/doc.go:23-24` and `internal/gitexec/transport.go:95`.
   - **Summary:** the docs claim an "unmounted origin directory reads as unreachable". When the origin URL is the mount point itself, that point still exists as an empty directory after unmount. `os.Lstat` then succeeds, and the code classes it as a refusal (`transport.go:88-89`). Only a repository below an unmounted mount point, or a removed directory, reads as unreachable.
   - **Evidence:** `transport.go:88-89`; the doc lines quoted.

**Tool calls**
1. Grepped repo docs, comments and fixtures for missing-path and transport wording, to find fixtures and docs the manifest lacked.
2. Read the remaining missing-path fixtures in `adoptremote`, `cleanupremove`, `destructivegaps` and `pendingteardown`, which the manifest did not cover.
3. Read the `adoptremote` unreachable rows and the users of `remoteEnvelopeBreakOrigin`, to see what they assert.
4. Read the cleanup remote-failure row and listed which pattern files mention transport, which my context lacked.
5. Read the transport and network wording in `PATTERN-pair-remote.md` and `PATTERN-daemon-wakeups.md`, to check the rule docs.
