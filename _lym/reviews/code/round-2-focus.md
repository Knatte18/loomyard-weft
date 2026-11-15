---
round: 2
focus:
  - "internal/pairteardown/teardown.go endOwnedProcesses and its callers in cancel.go and skip.go: does a cancel, skip or teardown on a beam's slug leave the beam's registered orch process (runproc.RoleOrch) running on the machine that still owns the beam?"
  - "internal/runproc/end.go endSubject: for the root subject owned by HubOwner and for a beam subject, which roles does it end, and are hub-wide roles (push, test-map, slack relay) and RoleTeardown children spared in every pass?"
  - "internal/orchengine/owner.go ownerTick and internal/orchcli/watch.go endSubjectProcesses: when the watcher reads a foreign owner while the beam's dobby is live, does the end cut short the dobby's own hand-over (dobbyengine handover.go handOver, ArchiveOwner)?"
  - "internal/orchengine/owner.go ownerTick: when the subject end fails persistently, is the orch strand still stopped on a machine that no longer owns the beam?"
  - "Subject-to-runs selection for ending a subject's processes: is it one exported function a caller outside orchcli can use, and does a test pin beam versus root selection, skipping the subject's own owner, and leaving an unreadable-subject owner alone?"
---

Targets the code the round-1 fixes touch.
