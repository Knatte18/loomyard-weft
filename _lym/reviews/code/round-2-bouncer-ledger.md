---
round: 2
ledger:
  - key: repeated-literals-not-named-constants
    rounds: [1]
    status: resolved
  - key: interrupt-leaves-dir-folders
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: benchmark-reports-omitted-without-deferral
    rounds: [1]
    status: resolved
  - key: dir-flag-registration-unpinned
    rounds: [1]
    status: resolved
  - key: failed-probe-cleanup-untested
    rounds: [1]
    status: resolved
  - key: go-build-work-folder-in-system-temp
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: link-build-environment-unpinned
    rounds: [1]
    status: resolved
  - key: git-variables-reach-git-workload
    rounds: [1]
    status: resolved
  - key: dir-bare-path-parsing
    rounds: [1]
    status: resolved
  - key: fresh-build-id-unpinned
    rounds: [1]
    status: resolved
  - key: throttle-count-sum-untested
    rounds: [1]
    status: resolved
  - key: no-dir-skip-not-recorded
    rounds: [1]
    status: resolved
---
Round 1 reviewed the new `tools/machinecost` tool.
Its findings covered named constants, folder cleanup, the environment of the timed children, command-line parsing and test coverage.

Round 2 resolved all of them but two, both about the probe's `go build` child:

- an interrupt during the build leaves the linker running while the probe removes its temp folder;
- the build still writes its cache and telemetry outside the probe's folders.

The missing `docs/benchmarks/` reports are settled by an added decision-record entry: they follow the operator's runs, outside this change.
