# Card 51 — accept-audit-evidence

**Create:**
- `internal/websterengine/suspect.go`
- `internal/websterengine/suspect_test.go`

**Edit:**
- `internal/websterengine/auditledger.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/webstercli/acceptaudit.go`
- `internal/websterengine/auditledger_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/acceptaudit_test.go`

**Uses:**
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/state.go`

**Intent:**
Fix PR #321 round-3 finding A's first three items: `AcceptPendingAudit` clears every pending correctness finding with no check, and `accept-audit` calls it with no argument, so any agent with Bash clears a run-exit halt by acknowledgement alone.
Accepting now needs evidence that each suspect path is back at the last recorded batch head.

`suspect.go` holds the path check this card and the next two share:

- `lastBatchHead(st *State) string`: the `Digest.HeadSHA` of the highest-numbered terminal batch, the integration key excluded, that carries one;
  "" when none does.
- `runStartCommit(st *State) string`: the `StartSHA` of the lowest-numbered batch record that carries one;
  "" when none does.
- `checkSuspectPaths(geom Geometry, st *State, base string, paths []string) (differing, unverifiable []string, err error)` sorts each path, resolved against `geom.WorktreeRoot` and link-resolved with `canonicalPath`, into one of three outcomes:
  - under `geom.PlanDir`: it differs when the file's SHA-256 is not `st.PlanFileHashes[<file name>]`, an absent file matching only an absent entry;
    with no `PlanFileHashes` at all it is unverifiable;
  - in the task worktree, not under its `_lyx` or `geom.ScratchDir`, and not git-ignored: it differs when `git diff --quiet <base> -- <path>` exits 1 or `git ls-files --others --exclude-standard -- <path>` lists it;
    with `base` empty it is unverifiable;
  - anywhere else (webster's state, reports or scratch files, a git-ignored path, a path outside the worktree): unverifiable, since nothing the run recorded says what it should hold.
  The error is a git probe's or a read's failure other than not-exist, and the check changes nothing.

In `auditledger.go`:

- Delete `dispositionAccepted`.
- `AcceptPendingAudit(st *State, geom Geometry) ([]PendingAuditFinding, error)` runs `checkSuspectPaths` over every pending finding's paths with `lastBatchHead(st)` as base.
  It refuses, mutating nothing, with a new exported sentinel `ErrAuditNotAcceptable` when any path differs, any path is unverifiable, or any finding has no path.
  The message names each differing path with the way forward `restore each with "git checkout <head> -- <path>" (delete a path the head does not hold), then re-run "lyx webster accept-audit"`,
  and each unverifiable path or pathless finding with the way forward `reset the branch to the run's start commit <sha> with git and run "lyx webster run --fresh"`, `<sha>` omitted when `runStartCommit` is empty.
  Otherwise it clears `PendingAuditFindings` and returns what it cleared, recording no disposition: a later run's audit covers a new Master session, whose finding identities never repeat these.
- The doc comment states the evidence rule.

In `runlevel.go`, the run-exit stuck reason and `pendingAuditFindingsError` keep their `accept-audit` clause, now `restore the named paths to the last batch head with git, then run "lyx webster accept-audit"`, and add, when any pending finding has no path, `a finding with no path clears only through "lyx webster run --fresh" after resetting the branch to the run's start commit`.

In `internal/webstercli/acceptaudit.go`, the verb calls `AcceptPendingAudit(st, c.geom)` and reports its error through `output.Err` before saving, so a refused call leaves `state.json` untouched and runs no fabric sync.
`Short` becomes `clear the pending run-exit audit findings once their paths are restored`, and `Long` says the verb checks every suspect path against the last batch head and refuses while any differs or cannot be checked.

`doc.go`'s run-exit paragraph states the rule.

Tests:

- `suspect_test.go` (integration-tagged, over `gitwrapNewScratchRepo`): a tracked file equal to the base → neither list;
  the same file edited in the worktree, or committed past the base → differing;
  an untracked new file → differing;
  a git-ignored file and a path under `_lyx/webster` → unverifiable;
  a plan card file whose hash matches `PlanFileHashes` → neither list, and one edited → differing;
  an empty base → a tracked path is unverifiable.
- `auditledger_test.go`: `TestAcceptPendingAudit_RecordsAcceptedAndClears` becomes `TestAcceptPendingAudit_ClearsWhenPathsMatchHead` over a scratch repo whose state records a done batch at HEAD: a pending finding naming an unchanged tracked file clears, and the ledger records nothing for it;
  `TestAcceptPendingAudit_RefusesDifferingPath`: the file edited → `ErrAuditNotAcceptable` naming the path and `git checkout`, the pending list unchanged;
  `TestAcceptPendingAudit_RefusesPathlessFinding`: a finding with no path → `ErrAuditNotAcceptable` naming `lyx webster run --fresh`.
- `runlevel_test.go`: the Master-`parent-write`-after-the-last-batch scenario calls `AcceptPendingAudit` before reverting and asserts `ErrAuditNotAcceptable` naming the path;
  after `git checkout` restores the file it accepts, and the next `Run` ends `done`.
- `acceptaudit_test.go`: the fixtures record a done batch whose `Digest.HeadSHA` is the fixture worktree's HEAD and a pending finding naming a tracked file there.
  `TestAcceptAuditCmd_AcceptsPendingThenIsIdempotent` asserts the cleared finding and no disposition;
  a new `TestAcceptAuditCmd_RefusesDifferingPath` edits the file, asserts a non-zero exit naming the path and `git checkout`, and `state.json` byte-identical;
  `TestAcceptAuditCmd_FabricSyncFailureNamesWayForward` drops its `accepted` disposition assertion and asserts the pending list is empty.

**ImpactSummary:** accept-audit clears pending run-exit findings only when every suspect path matches the last batch head or the run-start plan hash, and refuses pathless or uncheckable findings toward run --fresh; the accepted disposition is gone.
