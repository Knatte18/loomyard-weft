---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "Added GOCACHE and XDG_CONFIG_HOME as proposed, and also wrote a telemetry mode file holding off. An empty config folder leaves the mode at local, where cmd/go starts a detached (Setsid) telemetry process in that folder, which can outlive the build and race the temp folder's removal."
---

# Round 2 fixer report

## Changes

Commit `3f0ab187a` (F1 and F2, which share `tools/machinecost/newbinary.go`):

- F1, fixed as proposed: `linkProbeProgram` starts the build with `exec.Command`, drops `build.Cancel` and its comment, and returns the context's error after a successful build.
  The error is wrapped as `build probe program: ...`.
- F2, fixed differently: the build gets `GOCACHE` and `XDG_CONFIG_HOME` inside the temp folder, with telemetry turned off by a mode file.
  The proposed change not made is the config folder with no mode file.
  Evidence, in the Go 1.26 source under `/usr/lib/go-1.26/src`:
  `cmd/go/main.go` calls `telemetry.MaybeParent()` for every subcommand,
  and `cmd/vendor/golang.org/x/telemetry/start.go` starts the sidecar when the mode is not `off` and the folder holds no upload token younger than a day.
- The doc comment states where the go command's folders are, that telemetry is off, and what a cancelled context does.

Commit `40a8ea654` (self-check):

- The `go env GOVERSION` query in `tools/machinecost/machine.go` takes the same config folder with telemetry off, through a helper both go commands use.
- The probe build clears `GOCACHEPROG`.

Commit `4fffaaba1` (gate findings):

- Rewrote the comments the gate flagged in `tools/machinecost/machine.go` and `tools/machinecost/newbinary.go`, one sentence per line.
  No code changed.

Reach gate note, no commit and no test added:

- The functions listed as reached by no mapped test in `.lym/gate/reach.json`, other than `main` and `renderReport`,
  are on `runProbe`'s path, which the tier 1 tests in `tools/machinecost/probe_test.go` run.
  This rests on reading the code; the reach map names none of those tests.
- `main` wraps `run`, which has its own test, with temp folder removal and the process exit.
  `renderReport` writes text only a person reads.

Tier 0 and tier 1 tests of `./tools/machinecost` pass after the last commit, and `go vet` is clean.
No test was added: neither finding is a coverage gap.
No run observed the interrupt behaviour or listed the files written outside the probe's folders; both rest on the review's observation and the Go source.

## Self-check

- fix-code-correctness, LOW, `GOCACHEPROG` survives into the build: fixed.
- fix-code-correctness, LOW, a version-manager shim named `go` could lose its config under the overridden `XDG_CONFIG_HOME`: disputed, see below.
- fix-consistency: no findings.
- fix-affected-set, MEDIUM, `go env GOVERSION` still writes telemetry to the user's config: fixed.
- fix-proposal, MEDIUM, the same `go env GOVERSION` gap: fixed, merged with the line above.
- fix-proposal, LOW, `GOCACHEPROG`: fixed, merged with the first line.

## Disputed

No review finding is disputed.

Self-check finding, fix-code-correctness, the shim and `XDG_CONFIG_HOME`: ground `every fix worse`.
Go takes its telemetry folder from the user config folder, so keeping the user's `XDG_CONFIG_HOME` for a shim puts the telemetry writes of F2 back.
Asking the shim for the real toolchain under the user's config runs a go command that writes there too,
since `cmd/go/main.go:103-105` in the Go source opens counters before it dispatches any subcommand.
A shim that fails this way fails the probe with the command's error.

## Deferred

None.

## Rulings

None.
