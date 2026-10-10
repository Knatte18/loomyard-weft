---
round: 3
ledger:
  - key: d12-lyxcwd-breaks-subpath-anchored-hubs
    rounds: [1]
    status: resolved
  - key: d2-marker-misses-hub-building-git
    rounds: [1]
    status: resolved
  - key: d7-buildwithldflags-drops-flags
    rounds: [1]
    status: resolved
  - key: d11-hub-constructors-bypass-tier-purity
    rounds: [1]
    status: resolved
  - key: d11-shared-template-contamination
    rounds: [1]
    status: resolved
  - key: d11-write-definition-incomplete
    rounds: [1]
    status: resolved
  - key: scope-omits-changed-packages
    rounds: [1]
    status: resolved
  - key: d13-allowlist-and-match-shape
    rounds: [1]
    status: resolved
  - key: d7-bound-overstates-agent-deny
    rounds: [1]
    status: resolved
  - key: d6-bound-omits-packages-without-main
    rounds: [1]
    status: resolved
  - key: d12-git-dir-bound-unstated
    rounds: [1]
    status: resolved
  - key: d12-second-local-dotgit-reader
    rounds: [1]
    status: resolved
  - key: d11-copyhub-scan-path-prefix
    rounds: [1]
    status: resolved
  - key: d8-quotes-goal-not-in-goal-section
    rounds: [1]
    status: resolved
  - key: d12-reader-test-import-cycle
    rounds: [2]
    status: resolved
  - key: d12-cwd-resolution-constraint-understated
    rounds: [2]
    status: resolved
  - key: d12-walkup-diverges-from-show-toplevel
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-proc-probes-duplicated
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: census-drop-column-unnamed
    rounds: [2]
    status: resolved
  - key: test-budget-list-incomplete
    rounds: [2]
    status: resolved
  - key: d12-hooks-path-spawn-removed
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
---

Rounds 1 and 2 raised twenty keys between them; round 1's fourteen were all resolved by round 2.
Round 2's import-cycle, `PATTERN-cwd-resolution` constraint, census-column and test-budget keys were not raised again and the round 3 focus verdicts record them as sound.

Two round 2 keys persist.
`d8-proc-probes-duplicated` is now BLOCKING `design`: D8's "only copy" premise is false because `loomcli` and `orchcli` keep their own watchdog recognizer and `/proc` probes.
`d12-walkup-diverges-from-show-toplevel` stays MEDIUM `design`: D12's repository test does not require the `commondir` target to resolve, where git refuses it.

Round 3 adds one key, `d12-hooks-path-spawn-removed`: D12's survival rule for `rev-parse` would drop `--git-path hooks`, which honours `core.hooksPath`.
