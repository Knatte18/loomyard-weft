---
round: 2
ledger:
  - key: prebuild-go-build-uncapped
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
  - key: copyhub-dirty-lyx-warp-unrecorded
    rounds: [1]
    status: resolved
  - key: tmuxkit-nested-kit-dir-socket-overflow
    rounds: [1]
    status: resolved
  - key: repo-toplevel-duplicates-lyxcwd
    rounds: [1]
    status: resolved
  - key: harness-prebuildlyx-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
  - key: gate-tagged-build-failure-untested
    rounds: [1]
    status: resolved
  - key: resumearming-test-asserts-absence
    rounds: [1]
    status: resolved
  - key: tmuxkit-tests-missing-parallel-comment
    rounds: [1]
    status: resolved
  - key: tmuxkit-package-comment-wording
    rounds: [1]
    status: resolved
  - key: file-headers-missing
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's BLOCKING (the prebuild `go build` outside the slot's `-p` cap) is fixed for the gate.
It survives in round 2 as a decision-class MEDIUM: the `cmd/testtiming` default mode still builds with go's default parallelism.
The harness build's `-p` cap is still not pinned by a test.
The three round-1 design MEDIUMs and the remaining round-1 LOWs and NIT did not recur and are resolved.
Round 2 adds one LOW: three files in multi-file directories carry no file header.
