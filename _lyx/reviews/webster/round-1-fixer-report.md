# Webster round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each, none pushed.
After the last fix, `go build ./...`, `go vet ./...`, `go test ./...`, the plan's integration test set and the smoke-tag vet all pass, and `gofmt -l` over the diff's Go files is empty.

## Fixed

- **F1** (`f7ea13b1c`): `TransientGitHubAPI`'s doc comment in `internal/shedengine/transient.go` now lists a network error, a 5xx or a timeout as qualifying, and says a rate limit never does, which matches `githubclient.IsTransient`.
- **F2** (`38cb96779`): every new or changed Go comment the diff added is reflowed to one sentence per line, with further breaks only at independent-clause boundaries.
  Where a changed line sat inside a legacy fixed-column paragraph (`stuckOrCancelled`, `LastStep`, `lastStepOf`, `driverPrompt`), only the changed sentences were reflowed and the untouched legacy lines kept their wrap.
- **F3** (`4797f7f82`): restored `result.Outcome != shuttleengine.OutcomeDone` in `internal/burlerengine/engine.go`, so gofmt is clean.
- **F4** (`36f581d75`): `githubclient.IsTransient` no longer treats a `*url.Error` with `Op: "parse"` as transient, nor one whose chain holds a TLS certificate failure (`x509.UnknownAuthorityError`, `x509.HostnameError`, `x509.CertificateInvalidError` or `*tls.CertificateVerificationError`).
  Its doc comment says so, and `transient_test.go` covers each case.
  The new imports are stdlib, so the package stays inside its leaf allowlist.
- **F5** (`e31fc7f56`): `TestDriverPrompt_TiesTeardownToDoneAndBusyAndParksElsewhere` now asserts that `driverRecordsCommand` appears in the prompt after the parking sentence begins.
- **F6** (`4518aa120`): the `--no-attach` help paragraph now states the general readiness rule first and the parked-driver case after it, rewrapped to match the rest of the paragraph.
- **F7** (`dec9d8ab6`): `runDriverSpawnAndWait`'s doc comment now covers the stale-marker removal before a spawn and the resume branch for a live, parked driver.

## Deferred

None.
