---
round: 1
ledger:
  - key: prebuild-go-build-uncapped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: copyhub-dirty-lyx-warp-unrecorded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: tmuxkit-nested-kit-dir-socket-overflow
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: repo-toplevel-duplicates-lyxcwd
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: harness-prebuildlyx-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: gate-tagged-build-failure-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: resumearming-test-asserts-absence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: tmuxkit-tests-missing-parallel-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: tmuxkit-package-comment-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings, one BLOCKING.
The BLOCKING finding is the prebuild `go build` running outside the slot's `-p` cap, in both the gate and the `cmd/testtiming` harness.
Three design MEDIUMs follow: `CopyHub` copies are never clean and the exception is unrecorded, the tmuxkit nested kit directory overflows the socket path, and `Repo.Toplevel` duplicates `lyxcwd` with no production caller.
The rest are two coverage gaps, two test-convention slips and one comment-wording nit.
