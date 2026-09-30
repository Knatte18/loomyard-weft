---
verdict: BLOCKING
rationale: "F1-F7 are all fixed and verified (build, vet, test, integration, smoke vet and gofmt all clean), but card 4's commit 3e5420f01 also mangled an untouched doc comment in internal/shuttleengine/startup_test.go:768 to \"TestStartup_RunDeadlineShorterThanWindow_NeverReadypins\", the same out-of-card whitespace drop as F3, which gofmt cannot catch and the fix round left in place"
---

# Webster-Review round 1 — Bouncer verdict

## Review range

`_lyx/shed/ly-drive-safe-repairs/status.json` gives `product.parent: main`.
The merge base is `676d9ec03`.
The subject is cards 1–14, the follow-up `5b9d1e72d`, and the seven round-1 fix commits (`f7ea13b1c` through `38cb96779`).

## Substrate

I re-ran the plan's `verify:` block at HEAD `38cb96779`.
`go build ./...`, `go vet ./...`, `go test ./...`, the integration test set and the smoke-tag vet all pass.
`gofmt -l` over the diff's Go files prints nothing.

## Round-1 findings: each fix checked against the diff

- **F1** (`f7ea13b1c`): `TransientGitHubAPI` in `internal/shedengine/transient.go` now reads "a network error, a 5xx, or a timeout", and it excludes rate limits with the same reason card 3 and `githubclient.IsTransient` give.
  Resolved.
- **F2** (`38cb96779`): I scanned all 204 added comment lines in the diff.
  None holds two sentences, and every mid-sentence break falls at a comma followed by a coordinating conjunction and its own clause.
  Legacy lines were left alone.
  Resolved.
- **F3** (`4797f7f82`): `result.Outcome != shuttleengine.OutcomeDone` is restored, and gofmt is clean.
  Resolved for that line, but see the new finding below.
- **F4** (`36f581d75`): `IsTransient` now excludes `Op: "parse"` and a certificate failure anywhere in the chain (`x509.UnknownAuthorityError`, `x509.HostnameError`, `x509.CertificateInvalidError`, `*tls.CertificateVerificationError`).
  The doc comment and `transient_test.go` cover each case, including a wrapped `*url.Error`.
  Resolved.
- **F5** (`e31fc7f56`): the test now asserts that `driverRecordsCommand` appears in `got[park:]`.
  Resolved.
- **F6** (`4518aa120`): the `--no-attach` paragraph gives the general readiness rule first and the parked-driver case second.
  Resolved.
- **F7** (`dec9d8ab6`): `runDriverSpawnAndWait`'s doc now names the removal of a stale marker before a spawn, and the resume branch.
  Resolved.

The round-1 report has no `## Focus departures` section, so there are no departures to ratify or reject.

## New finding

### [LOW] Card 4 corrupted an untouched test doc comment

**Location:** `internal/shuttleengine/startup_test.go:768`
**Issue:** Commit `3e5420f01` (card 4) rewrote the line `// TestStartup_RunDeadlineShorterThanWindow_NeverReady pins the run-deadline-anchoring decision: a` as `// TestStartup_RunDeadlineShorterThanWindow_NeverReadypins ...`, dropping the space after the function name.
Card 4 only adds `NotStarted` assertions to this file and never asks for this line to change.
This is the same stray whitespace drop as F3, in the same commit, but it sits in a comment, so gofmt cannot flag it.
The doc comment no longer starts with the function name followed by a space, which Go's doc-comment convention and the repository's comments both expect.
F3's fix restored only the `burlerengine` line.
I compared every `-`/`+` line pair in each commit of the range with whitespace ignored, and this is the only remaining whitespace-only change.
**Fix:** Restore the space: `// TestStartup_RunDeadlineShorterThanWindow_NeverReady pins the run-deadline-anchoring decision: a`.

## Checks that found no further defect

- **Transient chain:** `stepLocked` marks only the producer-error arm, and every other persist writes an empty class through `persist`.
  `transientFailure` checks for cancellation before it classifies.
  `gitrepo`'s push paths wrap `*gitexec.GitError` with `%w`, so the classifier sees it.
  `subcommand` skips `-c push.autoSetupRemote=true` correctly.
- **Resume path:** `resumeParkedDriver` retries only on `ErrPaneNotReady`, stops after 20 attempts, and removes the marker only after a verified send.
  Each failure return in `runDriverSpawnAndWait` releases `bootstrapLock`, and a successful resume falls through to the existing handover.
- **Per-card mechanical check:** every card group is `Create` or `Edit`, so no `Rename` AST script or `Delete` `assert-no-callers` check applies.
- **Documentation:** the Transient Stop Invariant and the new `transient` envelope key are in `CONSTRAINTS.md`.
  `docs/overview.md` describes the resume, `binary_changed` and `shedtransient`, and it landed in the same commits as the behaviour.
