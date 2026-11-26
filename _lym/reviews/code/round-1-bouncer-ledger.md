---
round: 1
ledger:
    - key: repeated-literals-not-named-constants
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: interrupt-leaves-dir-folders
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: benchmark-reports-omitted-without-deferral
      rounds:
        - 1
      status: open
      class: decision
      severity: MEDIUM
      fix: design-wrong
    - key: dir-flag-registration-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: failed-probe-cleanup-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: go-build-work-folder-in-system-temp
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: link-build-environment-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: git-variables-reach-git-workload
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: dir-bare-path-parsing
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: fresh-build-id-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: throttle-count-sum-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: no-dir-skip-not-recorded
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, over the new `tools/machinecost` tool.
Its findings fall into four groups:

- the repo's named-constant rule;
- cleanup of the folders the probe makes, on interrupt and on failure;
- the environment that reaches the timed `go build` and git children;
- command-line parsing and test coverage gaps.

The omitted `docs/benchmarks/` measurement reports are a `decision` item for the run's parent, not for the fixer.
