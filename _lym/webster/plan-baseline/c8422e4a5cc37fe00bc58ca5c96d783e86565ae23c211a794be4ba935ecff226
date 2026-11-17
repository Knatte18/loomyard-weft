# Card 1 — fixtries-package

**Create:**
- `internal/fixtries/doc.go`
- `internal/fixtries/testmain_test.go`
- `plan:internal/fixtries#FileName` -> `const FileName = "fix-tries.json"`
- `plan:internal/fixtries#Read` -> `func Read(dir string) (int, error)`
- `plan:internal/fixtries#Write` -> `func Write(dir string, count int) error`
- `plan:internal/fixtries#TestReadWrite` -> `func TestReadWrite(t *testing.T)`

**Edit:**
- `internal/adoptcli#failedEnd`
- `internal/adoptcli#greenEnd`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#fixFixture.tries`
- `internal/adoptcli#fixFixture.setTries`
- `internal/adoptcli#TestGreenEnd`
- `internal/adoptcli#TestRunCheck`

**Delete:**
- `internal/adoptcli/holds.go`
- `internal/adoptcli/holds_test.go`

**Uses:**
- `internal/adoptcli/doc.go`
- `internal/pauserecord/testmain_test.go`

**Intent:** Move the reader and writer of the per-subject `fix-tries.json` out of `internal/adoptcli` into a new package `internal/fixtries`, so the upgrade's first read in `internal/dobbyengine` (card 22) reads the try count without importing `internal/adoptcli`, which imports `internal/dobbyengine`.

`fixtries.Read(dir)` and `fixtries.Write(dir, count)` are today's `readTries` and `writeTries` bodies, exported unchanged:
`Read` returns 0 with no error for an absent file and creates nothing; `Write` records the count through `state.UpdateJSON`, keeping keys it does not declare.
`FileName` replaces `triesFileName`.
The package imports only `internal/state`, and its caller hands it the subject's directory, `filepath.Join(fabricengine.AdoptScratchDir(hub), subject)` as adoptcli's `subjectDir` builds it.
`doc.go` says what the file records, who calls the package (adopt's fix loop, green clear and `ResetTries`, and the upgrade's first read) and its import rule.

adoptcli's callers switch to `fixtries.Read` and `fixtries.Write`: `failedEnd`, `greenEnd`, and the `ResetTries` closure in `hubDepsAs`.
The test helpers and tests that call the old functions or read `triesFileName` switch too: `fixFixture.tries`, `fixFixture.setTries`, `TestGreenEnd` and `TestRunCheck`.
`holds.go` and `holds_test.go` are deleted; `TestTriesStore`'s two rows move into `TestReadWrite` (absent file reads 0 and creates nothing; a written count reads back), a tier0 test.
`internal/fixtries/testmain_test.go` is untagged, the package's `TestMain` support file mirroring `internal/pauserecord/testmain_test.go`.
`adoptcli/doc.go` names the file, not the Go functions, so it stays as is.

**ImpactSummary:** Pure relocation: three adoptcli call sites and four test members switch package; no behavior or file format changes.
