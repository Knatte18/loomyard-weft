---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/drive.go:276
    summary: The Park seam's record commit and the CommitRecords closure have no test; only non-nil is asserted
    fix: In internal/battencli, add a test (extend driverdeps_test.go or add a row-driven test beside it) that builds the driver deps with c.recordsDeps stubbed to a recorder, then calls deps.HandBack.Park with the stub committing successfully and again with Commit returning an error; assert in both cases that the commit was attempted and that the park marker (shedrun.ParkMarker) exists afterwards. In the same test call deps.CommitRecords() and assert the recorded commit message is the text batten, run records for, and the slug.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/shedrun/paths.go:dirHoldsFile
    summary: dirHoldsFile dropped the IsDir guard of the check it replaced, so a driver path that is a plain file is now staged
    fix: In dirHoldsFile, call os.Stat(dir) first and return false when it errors or the result is not a directory, as loomcli's holdsFile does, so DriverRecordsRel keeps the old rule that an absent, non-directory or file-less path is omitted.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shedrun/paths.go:dirHoldsFile
    summary: dirHoldsFile duplicates loomcli's holdsFile almost line for line
    fix: Export one helper from shedrun (for example HoldsFile, carrying the IsDir guard from F2) and make loomcli's holdsFile in internal/loomcli/wiring.go:172 call it, or replace its remaining callers (wiring.go reviews, durable and webster checks) with the shedrun helper and delete the loomcli copy.
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: internal/sheddriver/drive.go:831
    summary: A done stop with friction failed writes its hand-back report after CommitRecords, so that report stays untracked
    fix: In finishDone, when stop.Friction is "failed", run CommitRecords after handBack has written the report (handBack returns before Park for DoneFrictionFailed), or call CommitRecords a second time after it; keep the error handling as it is for the first call.
---

### [MEDIUM] Park commit and CommitRecords closure untested

**Location:** internal/battencli/drive.go:276, internal/battencli/drive.go:327
**Issue:** The hand-back commit in the Park seam, which the focus names as the behavior to settle ("Park still parks on every commit failure"), is reached by no test.
`driverdeps_test.go` checks only that `HandBack.Park` and `CommitRecords` are non-nil, and the `run_test.go` stub commits nothing.
Removing the commit, or turning its warning into a returned error that skips the park marker, would fail no test.

### [LOW] dirHoldsFile accepts a plain file

**Location:** internal/shedrun/paths.go:dirHoldsFile
**Issue:** The inlined `holdsFile` returned false for a path that is not a directory.
`filepath.WalkDir` on a regular file calls the callback once with a non-directory entry, so `dirHoldsFile` returns true and the file is staged.
The effect is harmless to git, but it departs from the emptiness rule the loom pathspec documents ("absent, unreadable, non-directory or file-less outcome omits the entry"), which the focus asked to keep identical.

### [LOW] Duplicate holds-a-file helper

**Location:** internal/shedrun/paths.go:dirHoldsFile
**Issue:** `dirHoldsFile` and `holdsFile` in `internal/loomcli/wiring.go` implement the same walk, and the second is still used for the reviews, durable and webster directories.
Two copies can drift, as F2 shows.

### [LOW] Done-with-friction-failed report left uncommitted

**Location:** internal/sheddriver/drive.go:831
**Issue:** `finishDone` runs `CommitRecords` and then, for friction failed, `handBack` writes a report under drive-reports and returns before `Park`, so no later commit of a done run need carry it.
The issue asks that nothing under `_lym` stays untracked.
The next status or records commit of the run does pick it up when one happens, so this is a narrow gap rather than a loss.

## Rejected

- Yield early-return skips `resumed = false` and the rules reset (lens:code-correctness): nothing reads either after a yield exit, and a resumed pass never enters the branch.
- A yield carrying `arbiter_attempt` no longer reaches `arbiterAttemptRefused` (lens:code-correctness): consistent with "a yield decides nothing", and the answer key asks for yields to save nothing.
- Unreadable directory swallowed by `dirHoldsFile` (lens:code-correctness): behavior carried over unchanged from the replaced check, and the pathspec must never fail the commit; no regression.
- A failed `CommitRecords` at a done stop fails the driver while Park only warns (lens:code-correctness): the existing seam contract, deliberate.
- `sheddriver/doc.go` optional-seam list stale (lens:code-affected-set): withdrawn by the lens; the text stays true.
- `commitRunRecords` passes `c.slug` where `runID` is meant (lens:code-correctness): identical for every slug but the literal self run id, the same as `readopt.go`.

## Scope against the Goal

The diff covers loom and batten alike through `DriverRecordsRel`, and answers the per-poll question: a yielded pass now saves no envelope.
Drive-reports, arbiter and tackler directories are staged by both drivers; attempt directories sit beneath them.
No `PATTERN` entry is touched, no `Pattern-Departure:` trailer is present, and the diff changes no stencil.
