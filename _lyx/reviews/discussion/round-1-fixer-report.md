# Fixer report: discussion round 1

Every finding was checked against the code and fixed.

## Fixed

- F1: D3 now makes curl's low-speed abort a transport failure, adding `operation too slow` to `IsTransportFailure`'s phrases.
  Confirmed against `internal/gitexec/transport.go`, whose `transportPhrases` lacks it.
  An acceptance criterion covers it.
- F2: D2 sets `maintenance.auto=false` beside `gc.auto=0` at both points, says why, and its bound names `git maintenance start` as outside lyx's control.
  D6's PATTERN entry, the config acceptance criterion and ledger Q2/Q3 name the new key.
  The `gitkit` and `hubforge` fixtures already set both keys.
- F3: D1 gains its lock discipline: `goGit()` is resolved before the lock, the read runs under the shared lock, the exclusive lock covers only the fingerprint check, `Reindex()` and the fingerprint update, and a wrapped read never calls another `Repo` method.
  Confirmed that `r.goGitMu` is a `sync.RWMutex` and that `goGit()` takes it exclusively.
- F4: D3's bound now says a timeout at a shed boundary gets the driver's one transient re-step and then hands back, so the worst case is two deadlines, and a best-effort push logs and continues.
  Confirmed against the shed driver stencil's Transient re-step rule.
- F5: three acceptance criteria were added: D2's keys applied at removal on a hub that lacks them, a `pull` transport failure classified true, and a remote command's low-speed environment with its abort classified true.
- F6: D6's gc PATTERN entry now states its carve-out from `PATTERN-fabric-destruction-chokepoint`.
- F7: Q6 now says the #447 expectation is resolved by an auto-mode assumption, and Open risks gained a line saying the expectation stays unmet until `lyx-cleanup` lands.
- F8: D2's rationale drops the object counts in favor of "above git's default auto-gc threshold", and its repack-under-another-pair sentence shrinks to "D2 bounds how often; D1 carries correctness", leaving the detail in Open risks.

## Disputed

None.

## Deferred

None.
