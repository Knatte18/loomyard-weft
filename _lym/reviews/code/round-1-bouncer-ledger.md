---
round: 1
ledger:
  - key: oom-step-stalls-watchdog-loop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: oom-kill-misattributed-in-shared-cgroup
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: oom-notice-names-earlier-kill
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: oom-notice-delivery-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: degraded-launch-warning-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-scope-job-seen-as-timeout
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: failed-post-loses-notice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: busctl-spawn-logged-at-debug
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hub-cap-value-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: memory-swap-max-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: keep-reason-misses-new-default
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: ended-process-test-cannot-tell-branch
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings: five MEDIUM `design` ones (F1 to F5), five LOW and two NIT.
The gating ones concern the OOM notice path: an inline, lock-blocking step in the watchdog loop, notice attribution and naming errors, and missing tests for the notice delivery and the degraded launch warning.
The keys map to the review's findings in order: F1 to `oom-step-stalls-watchdog-loop`, F2 to `oom-kill-misattributed-in-shared-cgroup`, F3 to `oom-notice-names-earlier-kill`, F4 to `oom-notice-delivery-untested`, F5 to `degraded-launch-warning-untested`, F6 to `failed-scope-job-seen-as-timeout`, F7 to `failed-post-loses-notice`, F8 to `busctl-spawn-logged-at-debug`, F9 to `hub-cap-value-untested`, F10 to `memory-swap-max-unasserted`, F11 to `keep-reason-misses-new-default`, F12 to `ended-process-test-cannot-tell-branch`.
