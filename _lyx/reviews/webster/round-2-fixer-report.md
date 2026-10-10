# Webster-Burler round 2 — fixer report

## Fixed

### F1 — start's merge_in_progress refusal has no spec row (`a413b4f89`)

Widened the loom section's `resume: merge in progress` row into `start/resume: merge in progress` in `contracts/specs/refusal-spec.md`.
Its trigger now names both verbs, and its way forward is "resolve the merge as the message names, then re-run the verb".
`TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` already reaches the `start` side and `TestResumeVerb` the `resume` side, so no test changed.

### F2 — bare-error refusals in the start path name no way forward (`b124b3554`)

`internal/loomcli/start.go`:

- `readRunStatus` is now the one status read for `currentProducer` and `runHaltedAtHandBack`.
  It logs a decode failure and reports the file as absent.
  A live driver strand without a park marker over an undecodable status file is now a silent success, the same deferral the spawn path already had, instead of a raw `state: decode failed` refusal.
  Refusing there would have no valid way forward, since the spec rules out hand-editing a status file.
- `statusReadFailedMessage` is the refusal for a non-decode status read failure on both the spawn path and the live-strand branch.
- `rerunMessage` appends the `re-run "<verb>"` clause to the merge-state probe error (with the caller's verb, so `resume` names `lyx loom resume`), the run-lock probe, the strand-table read, the retiring-strand removal and the stale park-marker removal.

`contracts/specs/refusal-spec.md`, loom section:

- `start: status read failed` widened to cover the live-strand branch.
- New `start: driver strand directory unreadable` row (strand-table read, retiring-strand removal).
- New `start/resume: merge state unreadable` row (merge-state probe error).
- The `raw I/O` row now lists `remove` and the park marker, which cover the stale-marker removal; the run-lock probe already fell under its lock file.

Tests, all rows in existing tables:

- `TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses`: the probe-error row asserts the retry clause.
- `TestRunDriverSpawnAndWait_LLMArm_ReleasesLockOnFailure`: the `strand read` row asserts the retry clause; new rows `run-lock probe`, `retiring strand removal` and `stale park marker removal` reach those refusals and assert it.
- `TestRunDriverSpawnAndWait_LiveDriverWithoutParkMarker`: new rows `undecodable status file is a no-op` and `unreadable status file names the failure and the retry`; the table now carries each refusal's kind and message substrings.

Verified: `lyx gate test ./internal/loomcli`, `lyx gate test --tags integration ./internal/loomcli` and `lyx gate test ./cmd/lyx ./contracts/...` green; `gofmt -l` and `lyx loom lint-comments` clean.
The tmux-tier `TestSmokeBootstrapLifecycle` was not run here, as tmux-tier runs are refused in this session.

## Disputed

None.

## Deferred

None.
